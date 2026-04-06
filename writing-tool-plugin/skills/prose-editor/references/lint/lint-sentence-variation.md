# Sentence Variation Analysis

**Dispatch tag:** `sentence-variation`

### Sentence Length Bands

| Band | Word count |
|---|---|
| Short | 1--8 |
| Medium | 9--17 |
| Long | 18--30 |
| Very long | 31+ |

### Detection Rules

- [ ] Flag when consecutive sentences settle into the same length band long enough to feel repetitive (~3+ consecutive as rough anchor)
- [ ] Flag as monotone when the same-band streak becomes pronounced (~5+ consecutive as rough anchor)
- [ ] Flag as lacking punch when a passage runs for a noticeable stretch without any short sentences (~500 words as rough anchor)
- [ ] Flag as choppy when a passage runs for a noticeable stretch without any long sentences (~500 words as rough anchor)

### Burstiness Metric [A][S]

Source: O'Sullivan 2025, Humanities and Social Sciences Communications / Nature.

Calculate sentence-length standard deviation (SD) across the passage:
- SD >= 8 words: healthy variation (human-like burstiness)
- SD 5--7 words: moderate -- may need attention
- SD < 5 words: low burstiness -- most robustly documented AI stylistic signal

Low SD is the single strongest statistical indicator of AI-generated text.

### Sentence Opener Patterns

Flag when sentences in a window share the same opener type often enough to become noticeable (~3+ in a 10-sentence window as rough anchor):
- Subject-verb ("He walked," "She said," "The team decided")
- Pronoun-led ("It was," "They were," "This meant")
- Adverbial ("Suddenly," "Meanwhile," "Unfortunately")
- Dependent clause ("When the...", "Although...", "Because...")
- Gerund ("-ing" opener)

### Passive Voice Thresholds

- Non-academic prose: flag when passive voice becomes a noticeable habit (~15%+ as rough anchor)
- Academic/technical: flag when passive voice dominates even for this register (~25%+ as rough anchor)
- Fiction dialogue: flag when passive voice intrudes on what should feel spoken (~10%+ as rough anchor)

### Paragraph Rhythm Assessment

- Check for paragraph length variation (similar logic to sentence bands)
- Flag when consecutive paragraphs feel uniform in length (~3+ paragraphs within 15% of each other as rough anchor)

### Verdict Scale

- **Dynamic**: strong length variation (SD ~8+ words), varied openers, mixed bands, paragraph variation present
- **Repetitive**: moderate length variation (SD ~5--7 words) or noticeable same-band streaks or opener repetition
- **Monotone**: low length variation (SD below ~5 words) or multiple flags from above categories (~3+ as rough anchor)

<!-- Source: Toolkit Module 2 (base); Paper B S2 (burstiness metric) -->
