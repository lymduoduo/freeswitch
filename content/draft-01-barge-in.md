# AI 语音机器人的 Barge-in 为什么总是做错

**状态**: 🟠 审稿中
**目标发布日期**: 2026-03-22
**字数**: ~2200字
**Meta Description**: AI voice agent barge-in 失败 90% 不是 LLM 问题——是 FreeSWITCH VAD 配置问题。这篇文章讲清楚为什么，和如何修。
**Keywords**: FreeSWITCH barge-in, voice agent interruption, VAD configuration, AI voice robot, SIP audio

---

你的 AI 语音机器人，用户每次想打断都打断不了——要等 bot 把这句话说完，才能开口。或者反过来，bot 刚说了两个字，用户的呼吸声就把它打断了。

两种情况都让人抓狂。

我在调这个问题的时候，第一个直觉是去看 LLM 层——是不是 stream 没有正确取消？是不是 context 传错了？排查了半天，什么都没找到。

后来才意识到：问题根本不在 LLM，在 FreeSWITCH 的 VAD。

这个教训值得单独写一篇文章，因为我见过太多人在同一个地方卡住。

---

## 什么是 Barge-in，为什么它很难做好

Barge-in 的定义很简单：用户在 bot 说话的过程中开口，bot 立刻停止，转而处理用户的输入。

体验好的 Barge-in 让对话感觉自然，像在和真人说话。体验差的 Barge-in 让用户觉得自己在操作 IVR——说话没人理，或者稍微动一下嘴就把系统搞乱了。

难做的原因在于，它涉及三层系统必须精确协调：

```
媒体层（FreeSWITCH）
  ↕  VAD 检测、音频播放控制
应用层（Go 服务）
  ↕  事件路由、状态管理
AI 层（ASR + LLM + TTS）
  ↕  转写、推理、语音合成
```

这三层的延迟叠加在一起，决定了用户从说话到 bot 停止的感知时间。任何一层处理不当，整体体验都会崩掉。

大多数工程师在搭第一版的时候，会从最熟悉的层开始——AI 层。先把 LLM 的流式响应做好，再加 ASR，再想办法在 ASR 有输出的时候发个中断信号。这个顺序本身没问题，但容易导致一个结果：媒体层被忽视了。

---

## 三种常见的错误实现

### 错误一：在 ASR 层检测打断，然后通知 LLM

做法是：持续监听 ASR 的流式输出，一旦有新的转写内容进来，就认为用户在说话，发中断信号给 LLM，取消当前的 stream 生成。

这个逻辑听起来合理，实际上有一个根本性的延迟问题。

ASR 的流式首字延迟，在主流服务商那里大概是 150–400ms。也就是说，用户已经开口说了将近半秒，你的系统才能感知到"有人在说话"。在这半秒里，bot 还在继续说，TTS 还在继续播，FreeSWITCH 还在继续推音频。

用户的感受：我明明说话了，bot 完全没反应。

更糟的是，ASR 转写出来的是文本，中间还要经过应用层的逻辑处理，再去取消 LLM stream，再去停止 TTS 请求，最后 FreeSWITCH 才能停止播放。整条链路加起来，轻松超过 1 秒。

这不叫 Barge-in，这叫"等 bot 自己说完"。

### 错误二：VAD 阈值配置太保守

FreeSWITCH 的 VAD（Voice Activity Detection，语音活动检测）默认配置是保守的。它需要一段"足够长、足够响"的声音才会认定用户在说话。

这个配置对于防止噪音误触发是合理的，但对 AI 语音机器人来说太慢了。

用户说一个简短的"等等"、"嗯"、"不对"，音量不大，持续时间不到 200ms——很可能根本触发不了 VAD。VAD 没触发，后面的一切都不会发生。

这种情况下，工程师通常会去查 ASR 层和 LLM 层，但其实连 VAD 都没触发，问题出在最底下。

### 错误三：中断信号发了，但 TTS 音频没停

这是最隐蔽的一类错误，也最让人崩溃，因为从日志上看一切正常。

你的应用层收到了打断信号，取消了 LLM stream，取消了 TTS 请求——但 FreeSWITCH 还在播放之前已经缓冲到本地的 TTS 音频。

用户看到 UI 上显示"AI 正在倾听"，但耳机里 bot 还在说话。

原因是：取消 TTS 请求只是停止了新音频的生成，但 FreeSWITCH 的播放队列里已经有数据了。你必须显式调用 `uuid_break` 来清空播放队列，这一步很多实现里漏掉了。

---

## 正确的架构：三层协调，媒体层优先

核心原则只有一个：**Barge-in 的触发必须发生在媒体层，不能发生在 ASR 层或 LLM 层。**

正确的信号流是这样的：

```
用户开口说话
    ↓
FreeSWITCH VAD 检测到语音活动（~20ms）
    ↓  同时发生两件事：
    ├─ 立即执行 uuid_break，停止当前 TTS 播放
    └─ 通过 ESL 向应用层发送 DETECTED_SPEECH begin-speaking 事件
    ↓
应用层 Go 服务接收事件（~5ms ESL 传递）
    ├─ 取消当前 LLM stream（如果还在生成）
    └─ 取消待发送的 TTS 请求（如果还在队列里）
    ↓
FreeSWITCH 开始接收用户音频，送入 ASR
    ↓
ASR 流式转写 → 完成后送入 LLM → 生成新响应
```

关键点在第一步：FreeSWITCH 的 VAD 检测比 ASR 快得多。VAD 只需要判断"有没有声音"，不需要理解内容，延迟可以做到 20–50ms。

用户开口后 20ms，TTS 就停了。这才是用户感知到的"bot 在听我说话"。

---

## FreeSWITCH VAD 配置实战

FreeSWITCH 里和 Barge-in 相关的 VAD 配置主要有两个层面。

**一、sofia profile 层面的 VAD 模式**

在 `sofia.conf.xml` 里：

```xml
<param name="vad" value="out"/>
```

`vad` 参数有三个值：
- `in`：对入方向（用户说话）做 VAD
- `out`：对出方向（bot 播放）做 VAD
- `both`：双向

对 AI 语音机器人来说，通常设置 `in` 或 `both`，让 FreeSWITCH 在用户说话时产生检测事件。

**二、通道级别的 VAD 灵敏度**

在 dialplan 里，针对具体通话设置：

```xml
<extension name="ai-agent">
  <condition field="destination_number" expression="^1000$">
    <!-- VAD 检测到多少 ms 的语音才认为是真实说话，而非噪音 -->
    <action application="set" data="vad_activity_timeout=150"/>
    <!-- 多少 ms 的静默才认为用户说完了 -->
    <action application="set" data="vad_silence_threshold=500"/>
    <action application="socket" data="127.0.0.1:8084 async full"/>
  </condition>
</extension>
```

默认值 `vad_activity_timeout` 通常是 300ms，对 AI 语音场景太慢了。我在项目里用的起点值是 150ms，实际效果要根据你的用户环境（背景噪音水平）进一步调整。

`vad_silence_threshold` 控制的是"用户停止说话"的判断，影响 ASR 何时认为一句话结束。这个值调太小会导致用户说到一半就被截断，调太大会让整体响应变慢。500ms 是一个相对安全的起点。

**如何调试 VAD 参数**

在 `fs_cli` 里实时观察 VAD 事件：

```bash
# 打开 FreeSWITCH 控制台
fs_cli

# 订阅语音检测事件
/event plain DETECTED_SPEECH

# 然后对着麦克风说话，观察事件触发时机和 Speech-Type 字段
```

你会看到 `begin-speaking` 和 `end-speaking` 事件。观察 `begin-speaking` 从你开口到触发的时间差，这就是你的 VAD 延迟。目标是让这个值 < 100ms。

如果发现 `begin-speaking` 经常不触发，大概率是 `vad_activity_timeout` 太高，或者 sofia profile 里的 `vad` 没有配置正确。

---

## ESL 事件处理：Go 实现

VAD 配置好了之后，Go 服务这边需要正确处理 `DETECTED_SPEECH` 事件。

```go
type VoiceAgent struct {
    eslConn       *esl.Connection
    streamCancel  context.CancelFunc
    mu            sync.Mutex
}

func (a *VoiceAgent) subscribeEvents() error {
    // 订阅语音检测事件
    return a.eslConn.Send("event plain DETECTED_SPEECH CHANNEL_HANGUP")
}

func (a *VoiceAgent) handleEvents(ctx context.Context) {
    for {
        select {
        case <-ctx.Done():
            return
        case event, ok := <-a.eslConn.Events:
            if !ok {
                return
            }
            switch event.GetHeader("Event-Name") {
            case "DETECTED_SPEECH":
                a.onDetectedSpeech(event)
            case "CHANNEL_HANGUP":
                a.onHangup(event)
            }
        }
    }
}

func (a *VoiceAgent) onDetectedSpeech(event esl.Event) {
    speechType := event.GetHeader("Speech-Type")
    uuid := event.GetHeader("Unique-ID")

    switch speechType {
    case "begin-speaking":
        // 用户开始说话
        // 第一步：立即停止 FreeSWITCH 的音频播放
        // uuid_break 清空播放队列，all 表示清空所有排队的 playback
        a.eslConn.ExecuteAsync("uuid_break", uuid, "all")

        // 第二步：取消当前正在进行的 LLM stream 或 TTS 请求
        a.mu.Lock()
        if a.streamCancel != nil {
            a.streamCancel()
            a.streamCancel = nil
        }
        a.mu.Unlock()

        // 第三步：开始接收新的 ASR 流（具体实现取决于你的 ASR 接入方式）
        a.startListening(uuid)

    case "end-speaking":
        // 用户说完了，停止 ASR 收音，触发 LLM 推理
        a.stopListeningAndProcess(uuid)
    }
}
```

几个实现细节需要注意：

**`uuid_break` 的参数**：第三个参数 `"all"` 表示清空所有排队的播放任务。如果不加 `all`，只会停止当前正在播放的那条，队列里的下一条还会继续播放。

**为什么用 `ExecuteAsync` 而不是 `Execute`**：`Execute` 是同步的，会等 FreeSWITCH 执行完再返回。在事件处理的热路径上用同步调用会阻塞后续事件的处理。`ExecuteAsync` 发出去就继续，执行结果通过 `CHANNEL_EXECUTE_COMPLETE` 事件异步回来。

**`streamCancel` 的锁保护**：`begin-speaking` 事件可能在多个 goroutine 里被处理（如果你的事件分发是并发的），直接操作 `streamCancel` 需要加锁，否则有竞态。

---

## 边界情况处理

把核心流程做对之后，有三个边界情况需要额外处理。

**背景噪音误触发**

办公室的空调声、键盘声，在 VAD 灵敏度高的时候会触发 `begin-speaking`，导致 bot 莫名其妙地停下来。

解决方式：引入一个简短的确认窗口。`begin-speaking` 触发后，不立即执行打断，而是等 50–80ms，确认这段时间内语音信号持续存在，再执行。这个延迟在人耳的感知范围内几乎不可察觉，但能过滤掉大部分噪音脉冲。

```go
case "begin-speaking":
    go func() {
        // 等待 60ms 确认这是真实语音，而非噪音
        time.Sleep(60 * time.Millisecond)
        // 再次检查是否还处于 speaking 状态
        if a.isSpeaking(uuid) {
            a.executeBargein(uuid)
        }
    }()
```

**快速连续打断**

用户说了"等"，停顿了一下，又说了"等一下"。这会触发两次 `begin-speaking`，产生两次打断流程，导致 ASR 启动了两次，LLM 接到了两条指令。

解决方式：在应用层加一个防抖，200–300ms 内的重复 `begin-speaking` 只处理第一次。

**Bot 说完最后一句话之后**

Bot 刚说完最后一句，用户的回应被当成了打断——但这时候 bot 其实说完了，这是正常的对话轮转，不是打断。

区分方法：在 bot 完成 TTS 播放（`CHANNEL_EXECUTE_COMPLETE` 事件，application 为 `playback`）之后，设置一个 200ms 的"冷却期"，在冷却期内忽略 `begin-speaking` 事件，当作正常的用户发言处理。

---

## 总结

Barge-in 做得好和做得差，对用户来说感知差异极大。做得好，用户觉得在和一个很懂他的人说话。做得差，用户感觉在操作 IVR，要等机器说完才能说话。

核心结论：

1. **Barge-in 必须在媒体层触发**，不能靠 ASR 层。FreeSWITCH VAD → `uuid_break` 这条链路的延迟要做到 50ms 以内。

2. **`vad_activity_timeout` 是关键参数**，从 150ms 开始调，不要用默认值。

3. **`uuid_break` 带 `all` 参数**，清空整个播放队列，不只是当前播放。

4. **三个边界情况都要处理**：背景噪音防抖、快速连续打断去重、bot 说完后的冷却期。

如果你现在的 Barge-in 体验不好，先做这件事：在 `fs_cli` 里订阅 `DETECTED_SPEECH` 事件，对着麦克风说话，看 `begin-speaking` 是否及时触发。如果这一步就有问题，后面的 ASR 和 LLM 调得再好也没用。

---

*Meta Description: AI voice agent barge-in 失败 90% 不是 LLM 问题——是 FreeSWITCH VAD 配置问题。这篇文章讲清楚为什么，和如何修。*

*Tags: FreeSWITCH, voice-agent, VAD, barge-in, Go, SIP, real-time-audio*
