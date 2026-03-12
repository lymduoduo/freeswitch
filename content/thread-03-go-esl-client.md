# 推文线程：用 Go 写健壮的 FreeSWITCH ESL 客户端

**配套文章**: draft-03-go-esl-client.md
**发布时间**: 文章发布后 1 小时内
**平台**: Twitter/X

---

## 线程正文（10 条）

---

**1/**

FreeSWITCH 凌晨重启了。

你的 Go 服务进程还活着，日志没有 error，监控绿的。

但电话 20 分钟没人接。

这是 ESL 客户端没做重连的经典症状 🧵

---

**2/**

ESL（Event Socket Library）是 FreeSWITCH 的外部控制接口。

两种模式：
- Inbound：你连 FreeSWITCH:8021，接收所有事件
- Outbound：FreeSWITCH 连你，每通电话一个连接

本文说 Inbound——问题最集中的地方。

---

**3/**

问题 1：没有重连逻辑

FreeSWITCH 重启、网络抖动、OOM 崩溃——连接断了之后，你的服务静默失效。进程活着，什么都不处理。

解法：指数退避重连

```go
delay := 2 * time.Second
for {
    if err := c.connect(ctx); err != nil {
        time.Sleep(delay)
        delay = min(delay*2, 30*time.Second)
        continue
    }
    delay = 2 * time.Second // 成功后重置
    c.readLoop(ctx)
}
```

---

**4/**

订阅事件时，永远不要用 `event plain ALL`

高并发下 FreeSWITCH 会把几百个内部事件全推过来，你的 channel 会被打满。

只订阅你实际处理的：

```
event plain CHANNEL_CREATE CHANNEL_ANSWER
  CHANNEL_HANGUP DETECTED_SPEECH
```

---

**5/**

问题 2：事件乱序

高并发下，CHANNEL_ANSWER 可能比 CHANNEL_CREATE 早到。

你去 map 里找这通电话的 session，查到 nil，然后 panic。

只在压力大的时候出现，很难复现，非常折磨。

---

**6/**

解法：每通呼叫独立串行队列

```go
type Session struct {
    events chan Event // 每个 session 自己的 channel
}

// 单 goroutine 消费，天然串行
func (s *Session) run() {
    for event := range s.events {
        s.handle(event)
    }
}
```

同一通电话的事件全进同一个 channel，单 goroutine 处理，顺序物理保证。

---

**7/**

问题 3：Session 清理的竞态

CHANNEL_HANGUP 触发清理 session，同时另一个 goroutine 在写通话记录，它还拿着 session 指针。

结果：panic 在凌晨 3 点。

---

**8/**

解法：done channel 表达生命周期

```go
type Session struct {
    done chan struct{}
    once sync.Once
}

func (s *Session) close() {
    s.once.Do(func() { close(s.done) })
}

func (s *Session) Do(fn func()) bool {
    select {
    case <-s.done:
        return false // 已结束，不执行
    default:
        fn()
        return true
    }
}
```

任何操作都用 `s.Do()` 包裹，session 结束后自动跳过。

---

**9/**

生产监控最重要的指标：

`esl_event_queue_len`（每个 session 的事件积压数）

队列积压说明这通电话的处理逻辑卡住了。

告警规则：
- > 32 → warning
- > 56 → critical

不做这个告警，卡住的通话会静默丢事件，你根本不知道。

---

**10/**

检查你现在的 ESL 客户端：

1. 连接断开后会自动重连吗？
2. 同一通电话的事件有顺序保证吗？
3. Session 清理和业务逻辑之间有保护吗？

三件事做好，能稳跑几个月。

全文（含完整代码）：[文章链接]

---

## LinkedIn 推广帖

**发布时间**: 文章发布后 24 小时

---

生产事故复盘：FreeSWITCH 定期重启配置后，我们的 Go 呼叫中心服务静默失效了 20 分钟——进程还活着，但所有进来的电话都没人处理。

根本原因：ESL 客户端没有重连逻辑。

这件事让我系统地整理了 Go ESL 客户端在生产环境里会出现的三类问题：重连缺失、事件乱序、Session 竞态条件。每一个在开发环境都不会暴露，但在生产里都会出现。

写成文章了，附完整 Go 代码实现。

[文章链接放评论区]

---

## 编辑检查清单

- [x] 每条推文 < 280 字符
- [ ] 代码块格式在 Twitter 上验证
- [ ] 文章链接替换为真实 URL
- [ ] 开头的生产事故细节：发布前确认可以对外说（无客户信息泄露风险）
