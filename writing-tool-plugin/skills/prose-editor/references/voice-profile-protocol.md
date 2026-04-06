# Voice Profile Protocol

Reference for creating, storing, and applying voice profiles during prose-editor review passes.

---

## 1. Voice Profile Creation -- Path A (Story-Architect Stub)

Path A creates a minimal voice profile from direct writer input during story-architect onboarding. The profile captures project-level defaults through friction questions.

### Friction Questions

1. **Emotional register**: "What's the dominant emotional register of your prose? (e.g., wry, earnest, detached, warm)"
2. **Formality level**: "How formal is your default sentence? (conversational with contractions / standard / literary / elevated)"
3. **Sentence rhythm**: "Do your sentences tend toward short and punchy, long and flowing, or deliberately varied?"
4. **Habitual constructions** (optional): "Any constructions you know you lean on? (e.g., fragments, parentheticals, em-dashes)"

### Stub Template Output

A Path A profile is written with `creation_path: stub` and populates only the fields the writer explicitly answered. Unanswered optional fields are omitted rather than guessed. The stub is sufficient for early-draft review but should be upgraded to prose-derived when enough sample text exists.

---

## 2. Voice Profile Creation -- Path B (Prose-Derived)

Path B derives a voice profile by analyzing submitted prose. This path produces a richer profile but requires writer confirmation before any characteristic is recorded.

### Analysis Protocol

Given a submitted text sample (minimum 1,500 words recommended), identify:

- **Sentence length tendency**: Measure average sentence length and variance. Classify as short (under 12 words average), mixed (12-20), long (20+), or varied (high variance across the range).
- **Formality register**: Assess vocabulary choices, sentence structure complexity, and tone markers. Classify as conversational, standard, literary, or elevated.
- **Diction range**: Evaluate word-level choices -- Anglo-Saxon vs. Latinate roots, monosyllabic vs. polysyllabic tendencies, domain-specific vocabulary density. Classify as plain, mid-register, ornate, or mixed.
- **Rhythm pattern**: Analyze clause structure, punctuation cadence, and paragraph-level pacing. Classify as staccato (short clauses, hard stops), flowing (long clauses, soft connectors), varied (deliberate alternation), or cadenced (rhythmic repetition patterns).
- **Characteristic constructions**: Identify recurring syntactic patterns -- fragments, rhetorical questions, parenthetical asides, em-dash interruptions, semicolon chains, list structures, delayed subjects, or other distinctive habits.
- **Contraction usage**: Count contractions against opportunities for contraction. Classify as frequent (80%+), occasional (30-80%), rare (under 30%), or never.
- **Humor tendency**: Assess presence and type of humor -- classify as none, dry (understatement, irony), frequent (regular comedic beats), or situational (humor tied to specific contexts or characters).

### Presentation Protocol

Each derived characteristic is presented to the writer individually for confirmation, modification, or rejection:

1. State the observed characteristic with supporting evidence from the sample.
2. Ask the writer to confirm, adjust, or reject the classification.
3. Record only confirmed or writer-modified values.
4. Rejected characteristics are excluded from the profile entirely.

No characteristic is written to voice-profile.md without explicit writer confirmation. The analysis is a starting point for conversation, not a verdict.

---

## 3. voice-profile.md Artifact Schema

```yaml
## Voice Profile
project: [slug]
created_by: writer-confirmed
created_at: [ISO date]
creation_path: stub | prose-derived

### Baseline Characteristics
- sentence_length_tendency: [short/mixed/long/varied]
- formality_register: [conversational/standard/literary/elevated]
- diction_range: [plain/mid-register/ornate/mixed]
- rhythm_pattern: [staccato/flowing/varied/cadenced]
- characteristic_constructions: [writer-confirmed examples]
- contraction_usage: [frequent/occasional/rare/never]
- humor_tendency: [none/dry/frequent/situational]

### Preserve List
[5-7 distinctive features that editing should NOT flatten]

### Watch List
[2-3 weaknesses that editing can improve WITHOUT changing voice]

### Provenance
status: writer-confirmed
last_updated: [ISO date]
```

### Field Definitions

**Preserve List**: Features the writer considers essential to their voice. Editing passes treat these as constraints -- flagging them for removal is an error. Examples: "sentence fragments for pacing," "parenthetical asides as narrative commentary," "Anglo-Saxon diction in action scenes."

**Watch List**: Patterns the writer acknowledges as weaknesses. Editing passes may flag these without risk of flattening voice. Examples: "overuse of 'just' as a hedge," "comma splices that don't serve rhythm," "filtering verbs in close POV."

**Provenance**: Tracks confirmation status. A profile with `status: writer-confirmed` has been reviewed characteristic-by-characteristic. Profiles generated but not yet confirmed carry `status: draft` and are used with reduced confidence during review passes.

---

## 4. Voice Comparison Protocol

prose-editor uses the voice profile during Pass 3 (Voice and Authenticity) to detect drift from the writer's established baseline.

### Comparison Procedure

1. **Load voice-profile.md** from the active project's artifacts directory.
2. **Analyze submitted text** using the same dimensions as the Path B analysis protocol.
3. **Compare against baseline** for each characteristic:
   - Match: No flag. The text is consistent with the profile.
   - Minor departure: Note the drift with the specific characteristic and passage location. Severity: low.
   - Significant departure: Flag with evidence from both the profile and the submitted text. Severity: medium.
   - Contradiction: A characteristic directly opposes the profile (e.g., elevated diction in a conversational-register profile). Severity: high.
4. **Check the Preserve List**: Any edit suggestion that would flatten a preserved feature is suppressed.
5. **Check the Watch List**: Known weaknesses found in the text are flagged with higher confidence.

### Character Voice Anchors

When the project includes character-specific voice anchors (defined during story-architect planning), the comparison protocol layers character voice over narrative voice:

- Narrative prose outside dialogue is compared against the baseline voice profile.
- Dialogue and deep-POV interiority are compared against the relevant character's voice anchor.
- Mismatches between character voice and narrative voice are expected and not flagged as drift.

### Multi-POV Handling

For multi-POV projects, the writer annotates POV sections during submission. Each annotated section applies the corresponding character's voice anchor for comparison. Sections without annotation default to the baseline narrative voice profile. Phase 1 documents this protocol; automated POV-annotation detection is not required.

### Psychic Distance Diagnostic

Psychic distance -- the perceived closeness between reader and character consciousness -- is a primary vector for voice drift that vocabulary and rhythm analysis alone cannot catch. The diagnostic uses Card's four-level scale:

**Level 1 -- Distant/Omniscient**
"The city had been at war for three years."
Narrator is fully external. No character interiority. Useful for orientation, establishing scope, and temporal transitions.

**Level 2 -- Cinematic**
"Kael walked through the market, stepping over broken flagstones."
External observation of a specific character. Actions and sensory details are rendered, but the reader has no access to thought or feeling. Useful for scene-setting and action sequences.

**Level 3 -- Light Penetration**
"Kael noticed the market seemed quieter than usual. Something felt wrong."
Character awareness filters the narration. Filter words ("noticed," "seemed," "felt") signal the narrator mediating between reader and character. Useful for building tension and introducing character perspective.

**Level 4 -- Deep Penetration**
"Too quiet. The market never went silent, not even during the plague. Where was everyone?"
Direct rendering of character experience with no narratorial filter. Sentence structure, vocabulary, and rhythm reflect the character's mind rather than the narrator's prose. Useful for emotional peaks, crisis moments, and intimate interiority.

### Psychic Distance Application

When voice drift is detected, map the psychic distance of flagged passages:

- Determine the distance level of the flagged passage.
- Determine the distance level of the surrounding context.
- Verify whether the shift in distance is intentional and serves the scene's emotional arc.

Most scenes follow a natural distance pattern: open at Level 2-3 for orientation, deepen to Level 4 for emotional peaks, and pull back to Level 2-3 for transitions. An unintentional drop from Level 4 to Level 2 mid-scene often reads as voice drift even when vocabulary and rhythm remain consistent -- the reader experiences a sudden emotional distancing that breaks immersion.

Distance shifts that serve narrative purpose (a character dissociating under trauma, a deliberate pull-back for ironic effect, a scope transition between scenes) are noted but not flagged as errors.

<!-- Source: Paper A S8, Card's 4-level psychic distance scale -->
