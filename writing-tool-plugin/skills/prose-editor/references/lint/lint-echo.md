# Echo & Repetition Detector

**Dispatch tag:** `echo`

### Word Echo Rules

- Same content word (noun, verb, adjective, adverb) appearing within a 3-sentence window
- Severity increases with proximity: same sentence > adjacent sentences > 3-sentence window
- Weight unusual/distinctive words more heavily than common ones

### Phrase Echo Rules

- Multi-word phrases repeated close enough to create an echo effect (~3+ word phrases within 500 words as rough anchor)
- Exact repetition: always flag
- Near-repetition (1 word changed): flag when the pattern recurs enough to feel like a tic (~3+ times as rough anchor)

### Structural Echo Rules

- Same syntactic pattern used consecutively enough to become rhythmically monotone (~3+ times as rough anchor):
  - "He [verbed]. He [verbed]. He [verbed]."
  - "[Subject] was [adjective]. [Subject] was [adjective]."
  - "There was [noun]. There was [noun]."
- Same sentence-opening word recurring noticeably in a short span (~3+ times in 5 sentences as rough anchor)

### Paragraph Opener Echo Detection

- First word/phrase of consecutive paragraphs matching
- First sentence structure of consecutive paragraphs matching
- Flag when consecutive paragraphs share the same opener pattern long enough to feel mechanical (~3+ consecutive as rough anchor)

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
