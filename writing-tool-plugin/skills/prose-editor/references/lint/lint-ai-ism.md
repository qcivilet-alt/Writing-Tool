# AI-ism Detection

**Dispatch tag:** `ai-ism`

### Diagnostic Principle

The signal is accumulation, not individual occurrences. Any single AI-associated word may appear in good human prose. Multiple patterns clustered together are diagnostic.

### AI Cliche Phrases Catalog

**Verbs** [A][S]
- delve, leverage, utilize, harness, streamline, underscore, navigate, foster, elevate, embark
- Additional zombie-diction verbs: illuminate, unravel, captivate, grapple, unveil, forge, bolster

**Adjectives** [A][S]
- pivotal, robust, innovative, seamless, intricate, multifaceted, meticulous
- Additional: nuanced, comprehensive, groundbreaking, cutting-edge, transformative

**Nouns** [A][S]
- landscape, realm, tapestry, synergy, testament, beacon
- Additional: catalyst, paradigm, cornerstone, nexus, underpinning

**Filler Phrases** [A][S]
- "It's important to note that..."
- "In today's fast-paced world..."
- "In the realm of..."
- "At its core..."
- "When it comes to..."
- "It's worth noting that..."
- "This serves as a testament to..."

### Zombie Diction Word List [A][S]

Source: Kobak et al. PubMed study; Max Planck Institute confirmation of 50%+ post-2022 spike.

High-confidence markers (3x+ frequency increase post-2022):
- delve, utilize, landscape, intricate, pivotal, multifaceted, nuanced
- underscore, realm, testament, tapestry, meticulous, comprehensive

### Structural Patterns

- [ ] Uniform paragraph length -- consecutive paragraphs that feel suspiciously similar in size (~within 20% of each other across 5+ paragraphs as rough anchor)
- [ ] Formulaic templates -- intro/body/conclusion with predictable topic-sentence-then-elaboration rhythm
- [ ] Numbered or bulleted lists where continuous prose would be more natural
- [ ] Concluding paragraphs that mechanically restate the introduction

### Participial Construction Overuse [A][S]

Source: PNAS study -- main clause + comma + -ing verb phrase appears at 2-5x human rate in LLM output.

- Flag when participial trailing phrases appear frequently enough to become a pattern (~2+ per 500 words as rough anchor)
- Pattern: "[main clause], [verb]-ing [rest of phrase]"
- Example: "She opened the door, revealing a vast chamber"

### Contrastive Reframe [A][E]

- Pattern: "It's not X, it's Y" or "This isn't about X -- it's about Y"
- Flag when the pattern recurs often enough to feel formulaic (~1+ per 1000 words as rough anchor)

### Rule-of-Three Abuse [A][E]

- Tricolon without cadence variation (all three elements same length/stress)
- Flag when identical-rhythm tricolons cluster enough to suggest mechanical repetition (~2+ per 1000 words as rough anchor)
- Distinguish from deliberate rhetorical tricolon (varies length, builds intensity)

### Em-Dash Overuse [A][E]

- The "ChatGPT dash" -- em dashes used for parenthetical asides
- Flag when em dashes appear frequently enough to form a pattern (~3+ per 1000 words as rough anchor)
- Especially diagnostic when paired with other AI-ism markers

### Elegant Variation / Synonym Cycling [A][E]

- Unnatural synonym rotation: "the building... the structure... the edifice... the construction"
- Referents swapped every mention without semantic reason
- Flag when synonym cycling becomes noticeable for the same referent (~3+ different synonyms within 500 words as rough anchor)

### Shallow Specificity [A][E]

- Sensory details that add no information: "gentle breeze" = "breeze-like breeze"
- Adjective-noun pairs where the adjective is the default association
- Examples: "quiet library," "bustling city," "crisp autumn air," "warm smile"

### Spectral Sensory Language [A]

- Ghostly imagery attached to the immaterial: "the weight of silence," "the texture of grief"
- Synesthetic metaphors used as filler rather than genuine perception
- Flag when these accumulate noticeably without narrative grounding (~2+ per 500 words as rough anchor)

### Absent Perspective [A][S]

Source: Jiang & Hyland 2025 -- engagement markers at 3x lower rate in AI text.

- Perpetual balance without genuine stance ("on one hand... on the other")
- Absence of first-person conviction, hedged assertions throughout
- Missing engagement markers: questions, directives, attitude markers, self-mention

### Retail Voice [A][E]

- Customer-service tone: overly helpful, anticipatory, devoid of sharp edges
- "I hope this helps!" energy in non-service contexts
- Preemptive qualification of every claim

### Manufactured Drama [A][E]

- Artificial tension injected into mundane contexts
- "But here's the thing..." / "And that changes everything"
- Stakes inflation without substance

### Hedging / Weasel Patterns

- "It is generally considered..." / "Many experts believe..."
- "This could potentially..." / "It may be worth considering..."
- Flag when hedging phrases accumulate noticeably in assertive contexts (~5+ per 1000 words as rough anchor)

### Tonal Flatness Indicators

- Absence of sentence-level surprise or personality
- Every paragraph at the same emotional register
- No humor, irony, self-deprecation, or edge anywhere in the piece

### Transition Overuse

- Flag when explicit transition phrases become a noticeable crutch (~2+ per 500 words as rough anchor)
- Watch for: "Furthermore," "Moreover," "Additionally," "In addition," "However,"
- Especially diagnostic when transitions are interchangeable (any could replace any other)

### Punctuation Tells

- Em dashes: flag when forming a pattern (~3+ per 1000 words as rough anchor; see dedicated section above)
- Semicolons: near-zero usage is an AI tell (humans use them occasionally)
- Exclamation marks: clusters suggest retail voice

### Severity Guidance

| Category | Isolated occurrence | Clustered (3+) | Pervasive |
|---|---|---|---|
| Zombie diction | Note | Flag | Critical |
| Structural patterns | Note | Flag | Critical |
| Participial trailing | Note | Flag | Critical |
| Filler phrases | Flag | Critical | Critical |
| Tonal flatness | -- | Flag | Critical |

### Contextual Examples

**Zombie diction example:**
  "The team leveraged innovative synergies to navigate the complex landscape."
  Signal: 4 zombie-diction words in one sentence (leveraged, innovative, synergies, landscape).
  Direction: Replace abstract nouns with what actually happened; replace vague verbs with specific actions.

**Participial overuse example:**
  "She opened the door, revealing a vast chamber. He stepped inside, noticing the dust. They moved forward, feeling the weight of centuries."
  Signal: 3 consecutive sentences using the same trailing participial construction.
  Direction: Vary sentence structure; let some actions stand alone without the participial tag.

**Hedging/weasel example:**
  "It could potentially be argued that this may represent a somewhat significant shift in how we might think about the issue."
  Signal: 5 hedging qualifiers in one sentence (could, potentially, may, somewhat, might).
  Direction: Identify the actual claim and state it; if uncertainty is the point, name what is uncertain and why.

**Tonal flatness example:**
  "The meeting was productive. The team discussed several important topics. Progress was made on key initiatives. Everyone left feeling positive about the direction."
  Signal: Every sentence at the same emotional register, no personality, no specificity.
  Direction: Find the one moment that mattered and describe it with detail; cut the rest.

<!-- Source: Toolkit Module 1 (base); Paper B S2 (enrichments) -->
