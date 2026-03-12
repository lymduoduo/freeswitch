# Article Writer Prompt

Use this prompt to draft full-length technical articles (800–3000 words).

---

```
[SYSTEM: Use the system-master.md prompt as base context]

Write a technical article based on the following brief:

## Article Brief

**Title**: [TITLE HERE]
**Target Length**: [SHORT: ~800w / MEDIUM: ~1500w / LONG: ~2500w]
**Primary Audience**: [RTC engineers / AI engineers / Tech founders]
**Content Pillar**: [RTC Deep Dives / AI Voice Agent / Go Engineering / Systems Thinking]
**Core Argument / Thesis**: [ONE SENTENCE: what is the main point this article proves?]
**Key Takeaway for Reader**: [What should the reader be able to DO or DECIDE after reading?]

## Key Points to Cover

1. [POINT 1]
2. [POINT 2]
3. [POINT 3]
[Add more as needed]

## Technical Details to Include

- Tools/technologies: [LIST]
- Code snippets needed: [YES/NO — describe what]
- Architecture diagrams to describe: [YES/NO — describe what]
- Specific numbers/benchmarks to mention: [LIST IF ANY]

## What to Avoid

- [Any misconceptions NOT to reinforce]
- [Competing approaches NOT to dismiss unfairly]

## Tone Notes

[Any specific tone adjustments from the default — e.g., "more opinionated than usual",
"keep it accessible for AI engineers who don't know SIP"]

---

## Article Structure Template

Use this structure unless a different structure serves the content better:

### Structure A — Problem/Solution (for tutorials and debugging walkthroughs)
1. The Problem (specific, concrete)
2. Why It's Harder Than It Looks (context and constraints)
3. The Solution (step-by-step with code)
4. Tradeoffs and Edge Cases
5. Summary / What to Remember

### Structure B — Architecture / Design Decision
1. The Decision to Make (frame the choice)
2. Option A — What It Is and When to Use It
3. Option B — What It Is and When to Use It
4. My Recommendation (with reasoning)
5. What I'd Do Differently in Hindsight

### Structure C — Deep Dive / Explainer
1. The Concept and Why It Matters
2. How It Actually Works (internals)
3. Common Misunderstandings
4. Practical Implications (what this means for your code)
5. Further Reading

---

## Quality Check (apply before finalizing)

- [ ] Does the first paragraph hook the reader with a problem or surprising statement?
- [ ] Is every claim backed by a specific example, number, or code snippet?
- [ ] Are all technical terms used correctly?
- [ ] Does the article end with a clear, actionable conclusion?
- [ ] Is the voice consistent throughout (direct, pragmatic, no fluff)?
```
