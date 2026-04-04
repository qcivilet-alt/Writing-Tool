# Diction & Specificity Audit

**Dispatch tag:** `diction`

### Weasel Word Catalog

- very, really, quite, rather, somewhat, fairly, pretty (as intensifier)
- basically, essentially, actually, literally (non-literal), virtually
- things, stuff, aspects, elements, factors (vague nouns)
- nice, good, bad, interesting, important (empty evaluatives)

### Weak Verb List

| Weak verb | Stronger alternatives (context-dependent) |
|---|---|
| is/was/were | (restructure sentence to eliminate be-verb) |
| get/got | obtain, receive, earn, acquire, become |
| make/made | construct, produce, craft, generate |
| do/did | perform, execute, accomplish, complete |
| put | place, position, set, deposit |
| have (as main verb) | possess, contain, hold, maintain |
| go/went | proceed, advance, travel, move |
| said (over-repeated) | vary with action beats instead of dialogue tags |

### Filler Phrase List

| Filler | Replacement |
|---|---|
| in order to | to |
| due to the fact that | because |
| at this point in time | now |
| in the event that | if |
| it is important to note that | (delete) |
| a large number of | many |
| has the ability to | can |
| in terms of | (restructure) |
| the fact that | that / (delete) |
| for the purpose of | to / for |

### Nominalization Patterns

Flag noun forms where the verb form is stronger:
- "made a decision" -> "decided"
- "conducted an investigation" -> "investigated"
- "gave consideration to" -> "considered"
- "reached an agreement" -> "agreed"
- "had a discussion about" -> "discussed"

### Missed Specificity Indicators

- Generic nouns where a specific term exists: "the building" (which building?)
- Approximate quantities where exact ones are available: "several" / "a number of"
- Abstract claims without grounding: "the results were significant"
- Telling without evidence: "it was beautiful" (what made it beautiful?)

### Adverb Audit Criteria

- Flag adverbs modifying strong verbs (redundant): "sprinted quickly"
- Flag adverbs propping up weak verbs (replace both): "walked slowly" -> "shuffled"
- Flag -ly adverbs in dialogue tags: "said quietly" -> action beat
- Flag when -ly adverbs accumulate noticeably in narrative prose (~4+ per 500 words as rough anchor)

### False Positive Note

Domain terms used precisely are NOT weak diction. "Factors" in a statistical context, "elements" in chemistry, "aspects" in philosophy -- these are precise, not vague. Always check domain context before flagging.

<!-- Source: Toolkit Module 3 -->
