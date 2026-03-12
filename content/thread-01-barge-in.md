# 推文线程：AI 语音机器人的 Barge-in 为什么总是做错

**配套文章**: draft-01-barge-in.md
**发布时间**: 文章发布后 1 小时内
**平台**: Twitter/X

---

## 线程正文（10 条）

---

**1/**

你的 AI 语音机器人 Barge-in 体验很差？

99% 不是 LLM 的问题。

是 FreeSWITCH VAD 的问题。

我调过这个坑，结果出乎意料 🧵

---

**2/**

先说 Barge-in 是什么：

用户在 bot 说话的过程中开口，bot 立刻停止，转而处理用户输入。

做得好：感觉像和真人对话
做得差：感觉像在操作 IVR，要等机器说完才能说话

---

**3/**

大多数工程师的实现路径：

监听 ASR 流式输出 → 有转写内容 → 发中断信号给 LLM

听起来合理，但有个根本问题：

ASR 首字延迟 150–400ms

用户已经开口说了半秒，你的系统才感知到"有人在说话"

---

**4/**

更糟的是，信号链路是这样的：

ASR 出文本 → 应用层处理 → 取消 LLM stream → 取消 TTS → FreeSWITCH 停播

每一步都有延迟。加起来轻松超过 1 秒。

这不叫 Barge-in，叫"等 bot 自己说完"

---

**5/**

还有一个更隐蔽的错误：

你取消了 TTS 请求，但 FreeSWITCH 播放队列里已经有缓冲的音频了。

结果：UI 上显示"AI 正在倾听"，用户耳机里 bot 还在说话。

必须显式调用 `uuid_break all` 才能清空播放队列。很多实现都漏了这一步。

---

**6/**

正确的架构：Barge-in 必须在媒体层触发，不能在 ASR 层。

```
用户开口（t=0）
  ↓ ~20ms
FreeSWITCH VAD 检测到语音
  ↓ 同时做两件事：
  ├─ uuid_break all（立即停止播放）
  └─ 向 Go 服务发 DETECTED_SPEECH 事件
  ↓
Go 服务取消 LLM stream + TTS 请求
```

VAD 不需要理解内容，只判断"有没有声音"，延迟 20–50ms。

---

**7/**

FreeSWITCH 的关键配置：

```xml
<!-- vad 参数：in / out / both -->
<param name="vad" value="in"/>

<!-- 通话级别 -->
<action application="set"
  data="vad_activity_timeout=150"/>
```

`vad_activity_timeout` 默认 300ms，对 AI 语音场景太慢。

从 150ms 开始，根据你的背景噪音环境调整。

---

**8/**

Go 里处理 DETECTED_SPEECH 事件：

```go
case "begin-speaking":
    // 先停播，再通知上层
    a.eslConn.ExecuteAsync(
        "uuid_break", uuid, "all")
    a.cancelCurrentStream()
```

注意：用 ExecuteAsync 不用 Execute。
同步调用会阻塞事件处理循环。

---

**9/**

三个必须处理的边界情况：

① 背景噪音误触发：`begin-speaking` 后等 60ms 再确认，过滤噪音脉冲

② 快速连续打断：200–300ms 内的重复事件只处理一次（防抖）

③ Bot 说完最后一句：播放完成后设 200ms 冷却，不当作打断处理

---

**10/**

如果你现在 Barge-in 体验不好，先做这一件事：

```bash
# fs_cli 里
/event plain DETECTED_SPEECH
```

对着麦克风说话，看 `begin-speaking` 是否及时触发。

如果这一步就有问题，ASR 和 LLM 调得再好也没用。

全文（含完整代码）：[文章链接]

你的 Barge-in 延迟现在是多少？

---

## LinkedIn 推广帖（单独发）

**发布时间**: 文章发布后 24 小时

---

在做 AI 语音机器人的过程中，Barge-in（用户打断）是我遇到的最典型的"在错误的层面解决问题"的案例。

大多数工程师花时间调 LLM 的流式取消逻辑，或者调 ASR 的响应速度——但问题实际上在 FreeSWITCH 的 VAD 配置层，比 ASR 早了整整一层。

核心教训：**实时语音系统的体验问题，往往发生在你不熟悉的那一层。**

具体排查方法和 Go 实现写在文章里了。

[文章链接放评论区]

---

## 编辑检查清单

- [x] 每条推文 < 280 字符
- [ ] 代码块在 Twitter 上格式是否正确（需要发布时验证）
- [ ] 文章链接替换为真实 URL
- [ ] 第 10 条的互动问题是否有人会回答（测试：我自己会回答这个问题吗？）
