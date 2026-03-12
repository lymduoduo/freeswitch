# Content Repurposing Prompt

Use this prompt to transform existing content into different formats across platforms.

---

```
[SYSTEM: Use the system-master.md prompt as base context]

Repurpose the following content into the specified target format.

## Source Content

**Original Format**: [Article / Thread / Talk / Notes / Code README]
**Original URL or Content**:
[PASTE CONTENT HERE OR DESCRIBE IT]

## Target Format

**Platform**: [Twitter/X / LinkedIn / Newsletter / Short Article / TL;DR Summary]
**Target Length**: [Specify]
**Audience Shift**: [Same audience / More technical / Less technical / Broader]

---

## Repurposing Rules by Target Format

### Article → Twitter Thread
- Extract the single most valuable insight as the hook tweet
- Turn each major section into 1–2 tweets
- Cut all explanatory context — assume reader will engage if intrigued
- End with a link to the full article

### Article → LinkedIn Post
- Open with a professional framing of the problem (not technical jargon)
- Focus on the business/career impact, not implementation details
- 150–300 words max
- End with a question to drive comments
- Include article link in first comment (not the post)

### Article → Newsletter Segment
- Write in a more personal, conversational tone
- Add context about why YOU cared about this topic this week
- Include 1–2 personal observations not in the original article
- End with a question to the reader

### Twitter Thread → Short Article
- Expand each tweet into a full paragraph
- Add code examples and architecture details cut for Twitter
- Add introduction and conclusion
- Polish transitions between sections

### Multiple Articles → Pillar Post
- Identify the common theme across source articles
- Create a comprehensive guide that references/links to originals
- Add a synthesis layer: what do all these pieces add up to?
- This becomes your evergreen SEO anchor content

### Talk/Notes → Article
- Start with the core argument from the talk
- Reconstruct the logical flow in written form
- Add code/diagrams that can't be conveyed verbally
- Cut audience-specific humor or context

---

## Quality Check for Repurposed Content

- [ ] Does the repurposed version stand alone? (Don't require reading the original)
- [ ] Is the voice consistent with the target platform?
- [ ] Did you add value beyond just reformatting? (New example, updated data, etc.)
- [ ] Are all technical claims still accurate in the new context?
- [ ] Is there a clear CTA appropriate for the platform?
```

---

## Repurposing Calendar Suggestion

For each major article (1500+ words):
1. **Day 0**: Publish article on blog
2. **Day 1**: Twitter thread (extract key insight)
3. **Day 3**: LinkedIn post (business angle)
4. **Day 7**: Newsletter segment (personal reflection)
5. **Week 4**: Review metrics, repurpose top performer into video script or talk outline
