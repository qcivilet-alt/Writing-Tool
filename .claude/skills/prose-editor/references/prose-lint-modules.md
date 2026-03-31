# Prose Lint Modules

Consolidated detection patterns for Pass 5 (Prose Quality). Load only sub-modules matching the active dispatch concern tag.

---

## Sub-Module 1 -- AI-ism Detection

**Dispatch tag:** `ai-ism`

### Diagnostic Principle

The signal is accumulation, not individual occurrences. Any single AI-associated word may appear in good human prose. Multiple patterns clustered together are diagnostic.

### AI Cliche Phrases Catalog

**Verbs** [O][S]
- delve, leverage, utilize, harness, streamline, underscore, navigate, foster, elevate, embark
- Additional zombie-diction verbs: illuminate, unravel, captivate, grapple, unveil, forge, bolster

**Adjectives** [O][S]
- pivotal, robust, innovative, seamless, intricate, multifaceted, meticulous
- Additional: nuanced, comprehensive, groundbreaking, cutting-edge, transformative

**Nouns** [O][S]
- landscape, realm, tapestry, synergy, testament, beacon
- Additional: catalyst, paradigm, cornerstone, nexus, underpinning

**Filler Phrases** [O][S]
- "It's important to note that..."
- "In today's fast-paced world..."
- "In the realm of..."
- "At its core..."
- "When it comes to..."
- "It's worth noting that..."
- "This serves as a testament to..."

### Zombie Diction Word List [O][S]

Source: Kobak et al. PubMed study; Max Planck Institute confirmation of 50%+ post-2022 spike.

High-confidence markers (3x+ frequency increase post-2022):
- delve, utilize, landscape, intricate, pivotal, multifaceted, nuanced
- underscore, realm, testament, tapestry, meticulous, comprehensive

### Structural Patterns

- [ ] Uniform paragraph length -- paragraphs within 20% of each other in word count across 5+ consecutive paragraphs
- [ ] Formulaic templates -- intro/body/conclusion with predictable topic-sentence-then-elaboration rhythm
- [ ] Numbered or bulleted lists where continuous prose would be more natural
- [ ] Concluding paragraphs that mechanically restate the introduction

### Participial Construction Overuse [O][S]

Source: PNAS study -- main clause + comma + -ing verb phrase appears at 2-5x human rate in LLM output.

- Flag: more than 2 participial trailing phrases per 500 words
- Pattern: "[main clause], [verb]-ing [rest of phrase]"
- Example: "She opened the door, revealing a vast chamber"

### Contrastive Reframe [O][E]

- Pattern: "It's not X, it's Y" or "This isn't about X -- it's about Y"
- Flag: more than 1 per 1000 words

### Rule-of-Three Abuse [O][E]

- Tricolon without cadence variation (all three elements same length/stress)
- Flag: 2+ identical-rhythm tricolons in 1000 words
- Distinguish from deliberate rhetorical tricolon (varies length, builds intensity)

### Em-Dash Overuse [O][E]

- The "ChatGPT dash" -- em dashes used for parenthetical asides
- Threshold: >3 em dashes per 1000 words
- Especially diagnostic when paired with other AI-ism markers

### Elegant Variation / Synonym Cycling [O][E]

- Unnatural synonym rotation: "the building... the structure... the edifice... the construction"
- Referents swapped every mention without semantic reason
- Flag: 3+ different synonyms for the same referent within 500 words

### Shallow Specificity [O][E]

- Sensory details that add no information: "gentle breeze" = "breeze-like breeze"
- Adjective-noun pairs where the adjective is the default association
- Examples: "quiet library," "bustling city," "crisp autumn air," "warm smile"

### Spectral Sensory Language [O]

- Ghostly imagery attached to the immaterial: "the weight of silence," "the texture of grief"
- Synesthetic metaphors used as filler rather than genuine perception
- Flag when 2+ appear within 500 words without narrative grounding

### Absent Perspective [O][S]

Source: Jiang & Hyland 2025 -- engagement markers at 3x lower rate in AI text.

- Perpetual balance without genuine stance ("on one hand... on the other")
- Absence of first-person conviction, hedged assertions throughout
- Missing engagement markers: questions, directives, attitude markers, self-mention

### Retail Voice [O][E]

- Customer-service tone: overly helpful, anticipatory, devoid of sharp edges
- "I hope this helps!" energy in non-service contexts
- Preemptive qualification of every claim

### Manufactured Drama [O][E]

- Artificial tension injected into mundane contexts
- "But here's the thing..." / "And that changes everything"
- Stakes inflation without substance

### Hedging / Weasel Patterns

- "It is generally considered..." / "Many experts believe..."
- "This could potentially..." / "It may be worth considering..."
- Threshold: >5 hedging phrases per 1000 words in assertive contexts

### Tonal Flatness Indicators

- Absence of sentence-level surprise or personality
- Every paragraph at the same emotional register
- No humor, irony, self-deprecation, or edge anywhere in the piece

### Transition Overuse

- Threshold: >2 explicit transition phrases per 500 words
- Watch for: "Furthermore," "Moreover," "Additionally," "In addition," "However,"
- Especially diagnostic when transitions are interchangeable (any could replace any other)

### Punctuation Tells

- Em dashes: >3 per 1000 words (see dedicated section above)
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

<!-- Source: Toolkit Module 1 (base); Paper B S2 (enrichments) -->

---

## Sub-Module 2 -- Sentence Variation Analysis

**Dispatch tag:** `sentence-variation`

### Sentence Length Bands

| Band | Word count |
|---|---|
| Short | 1--8 |
| Medium | 9--17 |
| Long | 18--30 |
| Very long | 31+ |

### Detection Rules

- [ ] 3+ consecutive sentences in the same length band -- flag as repetitive
- [ ] 5+ consecutive sentences in the same band -- flag as monotone
- [ ] No short sentences in a 500-word stretch -- flag as lacking punch
- [ ] No long sentences in a 500-word stretch -- flag as choppy

### Burstiness Metric [O][S]

Source: O'Sullivan 2025, Humanities and Social Sciences Communications / Nature.

Calculate sentence-length standard deviation (SD) across the passage:
- SD >= 8 words: healthy variation (human-like burstiness)
- SD 5--7 words: moderate -- may need attention
- SD < 5 words: low burstiness -- most robustly documented AI stylistic signal

Low SD is the single strongest statistical indicator of AI-generated text.

### Sentence Opener Patterns

Flag when 3+ sentences in a 10-sentence window share the same opener type:
- Subject-verb ("He walked," "She said," "The team decided")
- Pronoun-led ("It was," "They were," "This meant")
- Adverbial ("Suddenly," "Meanwhile," "Unfortunately")
- Dependent clause ("When the...", "Although...", "Because...")
- Gerund ("-ing" opener)

### Passive Voice Thresholds

- Non-academic prose: flag at >15% passive constructions
- Academic/technical: flag at >25%
- Fiction dialogue: flag at >10%

### Paragraph Rhythm Assessment

- Check for paragraph length variation (similar logic to sentence bands)
- Flag 3+ consecutive paragraphs of near-identical length (within 15%)

### Verdict Scale

- **Dynamic**: SD >= 8, varied openers, mixed bands, paragraph variation present
- **Repetitive**: SD 5--7 or 2+ same-band streaks or opener repetition
- **Monotone**: SD < 5 or 3+ flags from above categories

<!-- Source: Toolkit Module 2 (base); Paper B S2 (burstiness metric) -->

---

## Sub-Module 3 -- Diction & Specificity Audit

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
- Threshold: >4 -ly adverbs per 500 words in narrative prose

### False Positive Note

Domain terms used precisely are NOT weak diction. "Factors" in a statistical context, "elements" in chemistry, "aspects" in philosophy -- these are precise, not vague. Always check domain context before flagging.

<!-- Source: Toolkit Module 3 -->

---

## Sub-Module 4 -- Cliche & Dead Metaphor Scanner

**Dispatch tag:** `cliche`

### Cliche Catalog

**Business/Corporate**
- move the needle, circle back, low-hanging fruit, deep dive, synergy
- at the end of the day, think outside the box, game-changer, paradigm shift
- hit the ground running, raise the bar, push the envelope, take it offline

**Marketing/Sales**
- cutting-edge, best-in-class, next-generation, world-class, state-of-the-art
- unlock potential, drive results, elevate your brand, seamless experience

**General Prose**
- tip of the iceberg, slippery slope, double-edged sword, silver lining
- stood the test of time, in the nick of time, a whole new world
- at the crack of dawn, calm before the storm, light at the end of the tunnel
- left no stone unturned, crystal clear, easier said than done

**Emotional/Literary**
- heart pounding, blood ran cold, tears streaming, knot in stomach
- breathe a sigh of relief, shiver down spine, weight of the world
- time stood still, deafening silence, bated breath

### Dead Metaphor Identification Criteria

A metaphor is "dead" when:
- The figurative origin is no longer perceived ("foot of the mountain")
- It functions as a stock phrase with zero imagery
- Replacing it with the literal equivalent loses nothing
- It appears in the first phrasing that comes to mind for the concept

### Redundancy List

- free gift, past history, advance planning, end result, final outcome
- close proximity, future plans, basic fundamentals, true fact, added bonus
- completely destroyed, totally unique, very essential, absolutely necessary

### Overused Intensifier List

- absolutely, totally, completely, utterly, extremely, incredibly
- literally (non-literal), unbelievably, insanely, wildly
- Threshold: >3 intensifiers per 500 words

<!-- Source: Toolkit Module 4 -->

---

## Sub-Module 5 -- Echo & Repetition Detector

**Dispatch tag:** `echo`

### Word Echo Rules

- Same content word (noun, verb, adjective, adverb) appearing within a 3-sentence window
- Severity increases with proximity: same sentence > adjacent sentences > 3-sentence window
- Weight unusual/distinctive words more heavily than common ones

### Phrase Echo Rules

- Any 3+ word phrase repeated within 500 words
- Exact repetition: always flag
- Near-repetition (1 word changed): flag if pattern repeats 3+ times

### Structural Echo Rules

- Same syntactic pattern used 3+ consecutive times:
  - "He [verbed]. He [verbed]. He [verbed]."
  - "[Subject] was [adjective]. [Subject] was [adjective]."
  - "There was [noun]. There was [noun]."
- Same sentence-opening word 3+ times in 5 sentences

### Paragraph Opener Echo Detection

- First word/phrase of consecutive paragraphs matching
- First sentence structure of consecutive paragraphs matching
- Flag at 3+ consecutive paragraphs with same opener pattern

### Exclusion List

Do NOT flag echoes of:
- Articles (a, an, the)
- Pronouns (he, she, they, it, I, we, you)
- Common prepositions (in, on, at, to, for, with, by)
- Conjunctions (and, but, or, so, yet)
- Domain-specific terms that must repeat for clarity
- Character names and proper nouns (in fiction)
- Deliberate anaphora or rhetorical repetition (verify intent from context)

<!-- Source: Toolkit Module 5 -->

---

## Sub-Module 6 -- Readability & Pacing Check

**Dispatch tag:** `readability`

### Readability Targets by Audience

| Audience | Target grade level (Flesch-Kincaid) |
|---|---|
| Young adult / adolescent | 6--8 |
| General adult | 10--12 |
| Professional / business | 10--12 |
| Academic / technical | 12--16 |

Note: These are targets, not ceilings. A few sentences above range is fine. Flag sustained deviation (5+ consecutive sentences 4+ grades above target).

### Hard-to-Read Sentence Criteria

Flag individual sentences that meet ANY of:
- 40+ words
- 3+ subordinate clauses
- 2+ parenthetical insertions
- Requires re-reading to parse (multiple negatives, deeply nested conditions)

Acceptable exceptions:
- Stylistic long sentences with clear parallel structure
- Quoted material or dialogue that is intentionally complex
- Technical definitions that require precision

### Pacing Analysis

**Drag indicators:**
- Consecutive paragraphs of description with no action or dialogue
- Information dumps (3+ paragraphs of exposition)
- Redundant restatement of already-established points
- Over-detailed transitions between scenes

**Rush indicators:**
- Major events compressed into single sentences
- Emotional beats given no space (grief, revelation, conflict resolved in one line)
- Scene transitions with no grounding in new setting
- Multiple plot points crammed into one paragraph

**Stall indicators:**
- Circular reasoning or restated arguments
- Characters (or narration) revisiting the same ground without progress
- Paragraphs that could be deleted with no information loss

### Show-Don't-Tell Criteria (Fiction/Creative Only)

Flag telling when showing would strengthen the prose:
- Emotion labels without behavioral evidence: "She was angry"
- Character trait declarations: "He was a kind man"
- Atmosphere labels: "The room felt eerie"
- Relationship summaries: "They were close friends"

Do NOT flag:
- Narrative summary bridging time gaps (acceptable telling)
- Character interiority where the POV character reflects on their own state
- Expository passages in non-fiction (telling is the mode)

### Paragraph Length Thresholds

| Format | Maximum recommended (words) |
|---|---|
| Digital / web | 150 |
| Print / longform | 200 |
| Academic | 250 |
| Dialogue-heavy fiction | 100 (interspersed with beats) |

Flag paragraphs exceeding the threshold for the target format. Single-sentence paragraphs used for emphasis are acceptable but flag if >3 appear in 1000 words (overuse dilutes impact).

<!-- Source: Toolkit Module 6 -->
