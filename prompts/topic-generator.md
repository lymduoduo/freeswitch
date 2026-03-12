# Topic Generator Prompt

Use this prompt to generate a batch of content topic ideas aligned with your pillars and audience.

---

```
[SYSTEM: Use the system-master.md prompt as base context]

Generate 20 content topic ideas based on the author's real project experience. These topics must
be grounded in ACTUAL work the author has done — not hypothetical scenarios.

## Author's Project Pool (draw from these)

1. FreeSWITCH + AI voice robot (streaming ASR/TTS, VAD, barge-in, media routing)
2. FreeSWITCH ESL + CTI call-center system (Go ESL client, event ordering, ACD queue)
3. FreeSWITCH H.264 video calling (SDP negotiation, AV sync, codec compat, bandwidth)
4. Simultaneous interpretation system (streaming ASR → translation → TTS, ultra-low latency)
5. End-to-end voice LLM integration (direct audio I/O, WebSocket + FreeSWITCH, no ASR/TTS layer)

## Topic Generation Rules

1. Each topic must be rooted in ONE of the 5 projects above — no generic topics
2. The "hook" must describe a REAL PAIN the reader has felt (not "learn about X")
3. Topics should NOT be beginner tutorials — assume the reader is a working engineer
4. Prioritize topics where the author has a UNIQUE angle (things most people get wrong, or
   problems that have no good existing content)
5. Mix across content types:
   - At least 4 "debugging walkthrough" topics (I saw X, diagnosed Y, fixed Z)
   - At least 4 "architecture decision" topics (why I chose X over Y, tradeoffs)
   - At least 3 "contrarian take" topics (challenge a common assumption)
   - At least 3 "deep technical explainer" topics (how something actually works)
   - Rest can be tutorials or case studies

## Output Format

For each topic, provide:
- **Title**: Specific, clickable title (not vague)
- **Source Project**: Which of the 5 projects this draws from
- **Pillar**: RTC Deep Dives / AI Voice Agent / Go Engineering / Systems Thinking
- **The Pain**: One sentence — what specific problem the reader is facing when they'd search this
- **My Unique Angle**: What makes this topic only credible coming from someone who's done it
- **Format**: Article / Thread / Tutorial / Opinion / Case Study
- **Difficulty to Write**: Easy / Medium / Hard
- **Repurpose Potential**: High / Medium / Low (can this become a thread + LinkedIn + newsletter?)

## Seed Ideas to Inspire (don't copy, use as direction)

- The barge-in problem: why most AI voice agents handle interruptions wrong
- ESL inbound vs outbound socket: when to use which, and the bugs each one hides
- Sentence boundary detection in streaming ASR: the real bottleneck in simultaneous interpretation
- H.264 SDP negotiation failures in FreeSWITCH: the 3 causes I've seen
- End-to-end voice LLM vs pipeline: a real latency comparison with numbers
- Go concurrency patterns for handling 500+ concurrent SIP sessions
- Why FreeSWITCH VAD settings matter more than your ASR model for voice agent quality
- The hidden costs of building your own SIP infrastructure vs Twilio (I've done both)

## Trend Context (use to sharpen relevance)

- End-to-end voice models (GPT-4o Audio, Gemini Live) are disrupting the ASR→LLM→TTS pipeline
- Real-time AI agents hitting production: latency and reliability are now the problems
- Go adoption growing in telecom/RTC backends (replace Python glue code)
- WebRTC + SIP interoperability pain is real and underdocumented
- Call-center AI (outbound/inbound) is a hot market, engineers need production-grade patterns
```

---

## Usage Notes

- Run monthly to replenish the content backlog
- After generating, score each with `ops/content-scorecard.md` (target: keep top 8)
- Move scored topics into `ops/publishing-calendar.md`
- Mark topics with "Repurpose Potential: High" — these get a full multi-format rollout
- Topics from Project 4 (同声传译) and Project 5 (端到端语音 LLM) are your rarest content —
  use them as anchor pieces (long articles) rather than quick threads
