---
guide_id: GUIDE.PROSE.AI_PATTERN_DETECTION
title: AI Pattern Detection
doc_type: leaf_guide
topic: prose
---

# AI Pattern Detection

A craft guide for understanding and identifying AI writing patterns in your prose.

---

## Why AI Patterns Emerge

Large language models produce text through statistical token prediction: each word is chosen based on probability distributions learned from training data. Safety training further smooths output toward agreeable, balanced, inoffensive prose. The result is text that is fluent but generic — it sounds like everyone and no one.

AI patterns are not errors in the traditional sense. They are statistical tendencies that cluster together, creating prose that reads as produced rather than authored. A single "delve" or "tapestry" in your writing is not a problem. Six of them in three paragraphs is a diagnostic signal.

---

## The Six Pattern Categories

### 1. Zombie Diction
**Why it's a tell:** Certain words spiked 50%+ in published text after 2022 (documented in Kobak et al., PubMed study). They are statistically associated with AI output, not because they're inherently bad, but because AI systems dramatically overuse them.

Watch for: delve, leverage, utilize, navigate, foster, elevate, embark, pivotal, robust, innovative, seamless, landscape, realm, tapestry, synergy, testament, beacon.

### 2. Structural Uniformity
**Why it's a tell:** AI generates consistently structured paragraphs because token prediction favors patterns seen frequently in training data. Human writing varies paragraph length, structure, and rhythm based on emphasis and pacing.

Watch for: paragraphs all roughly the same length, formulaic topic-sentence-then-support structure, every section opening the same way.

### 3. Hedging and Qualification
**Why it's a tell:** Safety training penalizes strong claims. AI systems add qualifiers reflexively, producing chains like "it seems like perhaps this could indicate" where a human would write "this shows."

Watch for: hedging chains (3+ qualifiers in one sentence), "it's worth noting," "it's important to consider," epistemic retreat from clear positions.

### 4. Tonal Flatness
**Why it's a tell:** AI lacks genuine emotional stakes. It can name emotions but rarely produces prose where the reader feels them. The result is a pleasant, even surface with no peaks or valleys.

Watch for: uniformly neutral tone, absence of urgency or conviction, "retail voice" (customer-service helpfulness), manufactured drama in mundane contexts.

### 5. Rhythmic Monotony
**Why it's a tell:** Human prose has high burstiness — sentence lengths vary dramatically based on meaning and emphasis. AI tends toward medium-length sentences with low standard deviation.

Watch for: sentence-length standard deviation below 8 words across a passage, 3+ consecutive sentences in the same length band, absence of fragments or long periodic sentences.

### 6. Absent Perspective
**Why it's a tell:** AI presents permanent balanced neutrality. Studies show AI-generated text has engagement markers at 3x lower rate than human writing. It presents "all sides" without ever committing to one.

Watch for: every paragraph presenting counterpoints, reluctance to take a position, "on the other hand" as a reflex rather than a genuine pivot, synonym cycling to avoid repetition.

---

## Distinguishing Real Patterns from False Positives

Not every instance of these patterns indicates AI generation. The key is **accumulation and clustering**.

**Legitimate uses of flagged patterns:**
- Domain terminology that happens to overlap with AI-ism lists (a business writer legitimately discussing "leveraging synergies")
- Intentional hedging in academic or scientific writing where epistemic caution is appropriate
- A writer's genuine stylistic preference for balanced, measured prose
- Deliberate structural uniformity for rhetorical effect

**When to investigate further:**
- 3+ categories co-occur in the same passage
- The patterns don't match the writer's established voice profile
- The patterns emerged after revision with AI tools
- The passage feels "produced" — fluent but impersonal

---

## When to Use prose-editor

This guide helps you self-identify AI patterns during your own revision. For detailed, systematic analysis:

- Use prose-editor with the `sounds-too-ai` concern tag for focused AI-pattern detection
- Use prose-editor with `full-review` for comprehensive analysis across all quality dimensions
- prose-editor provides line-level findings with severity ratings and craft-based diagnostics

**The goal is not AI-pattern avoidance — it's finding your own voice.** Prose that sounds like AI is prose that sounds like nobody. The fix is never cosmetic (swapping "delve" for another word) — it's having something specific to say and the craft to say it precisely.
