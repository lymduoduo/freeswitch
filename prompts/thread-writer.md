# Thread Writer Prompt

Use this prompt to write Twitter/X threads optimized for engagement and technical depth.

---

```
[SYSTEM: Use the system-master.md prompt as base context]

Write a Twitter/X thread based on the following brief:

## Thread Brief

**Core Topic**: [TOPIC IN ONE LINE]
**Thread Type**: [Choose one below]
**Hook Style**: [Choose one below]
**Target Length**: [5 tweets / 8 tweets / 12 tweets]

---

## Thread Types

- **Explainer**: Break down a complex technical concept into digestible tweets
- **Contrarian Take**: Challenge a widely-held belief with evidence
- **Debugging Story**: Walk through solving a hard problem (narrative arc)
- **Comparison**: Evaluate two approaches head-to-head
- **Lessons Learned**: Share hard-won insights from a project or failure
- **How-to**: Step-by-step guide to doing something specific

---

## Hook Styles

- **Problem Hook**: Start with a pain the reader recognizes
  Example: "If your FreeSWITCH calls are dropping randomly and you can't figure out why..."

- **Contrarian Hook**: Start with a statement that challenges assumptions
  Example: "Most teams building AI voice agents are optimizing for the wrong thing."

- **Result Hook**: Start with an outcome and promise to explain how
  Example: "I reduced our voice agent latency from 800ms to 180ms. Here's exactly what I changed:"

- **Mystery Hook**: Pose a question or puzzle that demands resolution
  Example: "Why does SIP work perfectly in your lab but fail in production? It's almost always this:"

---

## Thread Structure Rules

1. **Tweet 1 (Hook)**: The entire thread must be worth reading based on this tweet alone
2. **Tweet 2 (Context)**: Why this matters / who this is for
3. **Tweets 3–N (Body)**: Each tweet = one clear point. No tweet is just a transition.
4. **Second-to-last tweet**: The most valuable, insightful, or surprising point
5. **Last tweet**: Clear CTA or summary + ask for engagement

## Format Rules

- Number tweets: "1/" ... "2/" or use "🧵" indicator on first tweet
- Each tweet max 280 chars (hard limit)
- Use line breaks within tweets for readability
- Use `code formatting` sparingly for commands/config
- One concept per tweet — ruthlessly trim

## Technical Accuracy Check

After drafting, verify:
- All technical claims are accurate (SIP codes, RTP specs, Go API correctness)
- Code snippets compile / are syntactically correct
- No oversimplifications that would make experts cringe
- Specific tool versions mentioned where relevant

---

## Example Brief (Fill this in)

**Core Topic**: Why AI voice agents fail when users talk over the bot (barge-in problem)
**Thread Type**: Explainer + How-to
**Hook Style**: Problem Hook
**Target Length**: 10 tweets
**Key Points**:
1. The problem: VAD (Voice Activity Detection) latency
2. How most agents handle it (badly)
3. The FreeSWITCH side: mod_vad configuration
4. The LLM side: streaming + interrupt signals
5. The architecture that actually works
```
