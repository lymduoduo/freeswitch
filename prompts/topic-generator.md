# Topic Generator Prompt

Use this prompt to generate a batch of content topic ideas aligned with your pillars and audience.

---

```
[SYSTEM: Use the system-master.md prompt as base context]

Generate 20 content topic ideas for a Go backend engineer specializing in FreeSWITCH, SIP/RTP,
call-center systems, and AI voice-agent architecture.

## Topic Generation Rules

1. Each topic must solve a SPECIFIC problem a real engineer would face
2. Topics should NOT be beginner tutorials (assume engineering competence)
3. Mix across these content pillars:
   - RTC Deep Dives (FreeSWITCH internals, SIP debugging, RTP issues)
   - AI Voice Agent Architecture (pipeline design, latency, barge-in, LLM integration)
   - Go Engineering (concurrency, networking, performance in RTC contexts)
   - Systems Thinking (tradeoffs, architecture decisions, debugging methodology)
4. Include at least 3 "contrarian take" topics (challenge common assumptions)
5. Include at least 3 "deep debugging" topics (walkthrough of solving a hard problem)

## Output Format

For each topic, provide:
- **Title**: The exact article/post title
- **Pillar**: Which content pillar it belongs to
- **Hook**: One sentence explaining why an engineer would click this
- **Format**: Article / Thread / Tutorial / Opinion / Case Study
- **Difficulty to Write**: Easy / Medium / Hard
- **Estimated Audience Interest**: High / Medium / Niche

## Context for Better Ideas

Current trends to consider:
- AI voice agents becoming production-grade (latency, reliability challenges)
- LLM function calling + real-time audio pipelines
- FreeSWITCH vs cloud providers (Twilio, Vonage) cost/control tradeoffs
- WebRTC + SIP interoperability challenges
- Go becoming more common in telecom/RTC backends

My recent work includes:
[ADD YOUR RECENT PROJECTS/PROBLEMS HERE]

My most successful past content was about:
[ADD YOUR TOP-PERFORMING CONTENT HERE]
```

---

## Usage Notes

- Run monthly to build a content backlog
- After generating, score each topic in `ops/content-scorecard.md`
- Move approved topics to `ops/publishing-calendar.md`
- Tag topics that can be repurposed across multiple formats
