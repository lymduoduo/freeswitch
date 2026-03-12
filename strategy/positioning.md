# 品牌定位策略 / Brand Positioning Strategy

## 核心定位 / Core Position

**English Prompt:**
> I am a Go backend engineer with deep hands-on experience in FreeSWITCH and real-time communications
> infrastructure. I've built AI voice robots integrated with FreeSWITCH, implemented call-center
> systems via ESL+CTI, shipped H.264 video calling, architected simultaneous interpretation pipelines,
> and integrated end-to-end voice LLMs. My edge is that I've shipped all five layers of the stack —
> SIP signaling, RTP media, AI inference, real-time audio pipelines, and the LLM prompt layer —
> in production systems, not toy demos.

**中文定位陈述：**
> 我是一名 Go 后端工程师，深度专注于 FreeSWITCH 与实时通信基础设施。我做过的项目横跨整个 AI 语音栈：
> FreeSWITCH 对接 AI 语音机器人、ESL+CTI 呼叫系统、H.264 视频通话、同声传译系统、以及端到端语音大模型接入。
> 我的核心优势是：这五类系统我都在生产环境真正跑过，不是 Demo。

---

## 真实项目经历 / Real Project Background

> 这部分是内容创作的"弹药库"——每个项目都是一篇文章、一条线程、一个案例。

### 1. FreeSWITCH 对接 AI 语音机器人
**技术核心**: FreeSWITCH 媒体处理 + ASR/TTS 流式接入 + VAD + Barge-in 处理

**有价值的内容角度**:
- 端到端延迟优化（SIP 建立 → AI 响应 → 音频回流的完整链路）
- Barge-in（用户打断）的正确实现方式，以及大多数人犯的错
- FreeSWITCH mod_audio_stream / UniMRCP 等接入方案的实际对比
- 生产环境中 VAD 误触发的排查过程

---

### 2. FreeSWITCH ESL 连接 CTI 实现呼叫系统
**技术核心**: Event Socket Library、事件驱动架构、呼叫控制、ACD 队列、CTI 集成

**有价值的内容角度**:
- ESL 的工作机制：inbound vs outbound socket 的选型逻辑
- 用 Go 实现健壮的 ESL 客户端（重连、事件序列化、并发控制）
- CTI 协议集成的坑：时序问题、状态机设计、竞态条件
- 呼叫中心系统架构：一个真实的设计决策记录

---

### 3. FreeSWITCH H.264 视频通话
**技术核心**: SDP 协商、视频编解码、RTP 视频流、带宽管理、转码

**有价值的内容角度**:
- FreeSWITCH 视频支持的真实现状（坑比你想的多）
- H.264 SDP offer/answer 协商失败的常见原因排查
- 视频通话的带宽与质量权衡：码率、帧率、分辨率的实战调优
- 视频 + 音频同步问题（AV sync）的底层原因

---

### 4. 同声传译系统
**技术核心**: 实时音频流 → ASR → 翻译 → TTS，极低延迟要求，多语言，流式处理

**有价值的内容角度**:
- 同声传译 pipeline 的架构设计：延迟 vs 准确性的核心权衡
- 流式 ASR 的分句策略：如何决定"何时喂给翻译层"
- 多语言 TTS 的速率匹配问题（目标语言说话速度不同）
- 这类系统的错误处理策略：降级、静音、重试的设计

---

### 5. 对接端到端语音大模型
**技术核心**: 直接音频输入/输出的 LLM（如 GPT-4o Audio / Gemini Live），无 ASR→LLM→TTS 的中间层

**有价值的内容角度**:
- 端到端语音模型 vs 传统 pipeline 的架构对比（延迟、成本、可控性）
- FreeSWITCH 如何接入 WebSocket 音频流与 LLM API 通信
- 端到端模型的新挑战：情绪控制、打断处理、函数调用
- 什么场景应该用端到端模型，什么场景仍然用 pipeline

---

## 差异化优势 / Differentiation

| 维度 | 泛 AI 工程师 | RTC 工程师（不做 AI）| 我的定位 |
|------|------------|-------------------|---------|
| SIP/RTP 理解 | 无 | 深 | 深 |
| AI 语音集成 | 浅（API 调用）| 无 | 深（做过 5 类系统）|
| 视频通话 | 无 | 部分 | 有（H.264 实战）|
| 同声传译 | 极少 | 无 | 有（完整 pipeline）|
| 端到端语音 LLM | 新兴，多数只接过 | 无 | 有（含架构对比经验）|
| 生产经验 | Demo 居多 | 有 | 有（含真实踩坑）|

**核心空缺**: 能把 FreeSWITCH 底层和 AI 语音前沿同时讲清楚的工程师，目前几乎没有。

---

## 一句话定位 / One-liner

**EN:** I've shipped AI voice robots, simultaneous interpretation, and end-to-end voice LLMs on top of FreeSWITCH — and I write about what actually breaks in production.

**中文：** 我在 FreeSWITCH 上跑过 AI 语音机器人、同声传译和端到端语音大模型——我写的是生产环境里真正会炸的东西。

---

## 受众感知目标 / Perception Goals

当目标读者看到我的内容时，他们应该感受到：
1. **"这人把我一直没敢碰的系统做通了"** — 同声传译、端到端语音 LLM，门槛极高
2. **"他踩过这个坑，我不用再踩一遍"** — 真实的排查过程比教程更有价值
3. **"这是我遇到这类问题第一个应该找的人"** — 建立不可替代的专家印象

---

## 竞争格局分析 / Competitive Landscape

- **泛 AI 技术博主**: 写 LLM 应用但不懂电话系统，深度不够
- **电信/VoIP 工程师**: 专业但不碰 AI，内容停在 2015 年
- **语音 AI 创业公司博客**: 有深度但是营销目的，有偏见
- **我的空缺**: FreeSWITCH 实战 × AI 语音全栈 × 没有利益关系的工程师视角

---

## 定位演进路径 / Evolution Path

```
Phase 1 (现在 0-6月)
  标签：FreeSWITCH × AI 语音的实战工程师
  内容：5 个项目的真实经验，踩坑总结，架构分析

Phase 2 (6-12月)
  标签：AI 语音基础设施架构师
  内容：系统设计框架，选型指南，深度对比

Phase 3 (12-24月)
  标签：实时 AI 通信领域的 Go-to 工程师
  内容：课程、咨询、社区、行业观点
```
