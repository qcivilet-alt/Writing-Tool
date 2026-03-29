# story-architect Skill — Implementation Plan

> **For agentic workers:** REQUIRED: Use superpowers:subagent-driven-development (if subagents available) or superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build the story-architect Claude skill — a structured story-planning coach with gates, friction engine, genre abstraction, and anti-ghostwriting guardrails — plus thin routing extensions for writing-guide.

**Architecture:** 8 new files across 2 skill directories, plus 1 KB stub and a minimal writing-guide SKILL.md append. story-architect has SKILL.md (~480 lines) orchestrating 5 reference files loaded on demand. writing-guide gets 2 new reference files for specialist routing and a 4-line load-condition append to its SKILL.md.

**Tech Stack:** Claude skills (markdown instruction files), JSON schemas for artifacts/escalation payloads, writing_mentor_kb integration.

---

## Context

The Writing Tool is a three-module ecosystem: writing-guide (existing hub), story-architect (this build), prose-editor (future). The spec (`docs/superpowers/specs/2026-03-28-writing-tool-design.md`, all 7 sections approved) defines story-architect as a structured story-creation workflow that helps writers move from raw ideas to a formalized story framework through 4 planning gates, a friction engine, and genre-adaptive character frames — without ever generating prose.

**Prerequisite resolved:** All 8 KB files referenced in the gate-to-KB mapping (spec Section 4.7) exist in the KB. The memoir-specific `GUIDE.CRAFT.MEMOIRIST_DISTANCE.md` referenced in Section 2 does NOT exist — this plan creates a stub (Task 3) and kb-gate-map.md documents the gap.

**Source verification:** All 6 primary sources are mapped to existing KB files — Weiland story structure (9 guides), Weiland scene structure (1 guide), Purdue OWL pitfalls (2 guides), Reedsy comparative frameworks (1 memo), Janice Hardy revision ripple (2 files), Weiland character arcs (2 guides). SOURCE_AUDIT.md and SOURCES_USED.md confirm attribution.

**Deferred to prose-editor spec:** AI-pattern avoidance guidance (detecting AI-sounding tropes, word choices, sentence structures in writer prose + building voice strength that resists AI patterns). This is a prose-review concern, not a planning concern. story-architect's anti-ghostwriting controls prevent the *tool* from generating prose; prose-editor will need a dedicated `GUIDE.PROSE.AI_PATTERN_DETECTION.md` (~150 lines, both detection and prevention) to coach *writers* on recognizing and avoiding AI-patterned output in their own work.

**Spec naming resolution:** Section 4.2 directory structure is canonical where Section 2 and Section 6.2 conflict:
- Characters: `characters/[character-slug].md` (one file per character)
- Chapters: `chapters/[nn]-[slug].md` (one file per chapter)
- Include `cross-character-relationships.md` and `continuity-ledger.md` per Section 4.2/6.2

---

## File Structure

```
~/.claude/skills/story-architect/          (NEW directory)
├── SKILL.md                               ~480 lines — main skill
└── references/
    ├── artifact-schemas.md                ~300 lines — field schemas, handoff JSON, escalation payloads
    ├── genre-vocabulary.md                ~100 lines — substitution tables by genre tag
    ├── character-frames.md                ~140 lines — 4 genre character frames
    ├── friction-patterns.md               ~240 lines — gap scan, triggers, validation, sprint mode
    └── kb-gate-map.md                     ~150 lines — KB loading per gate, required fields, severity

~/.claude/skills/writing-guide/references/ (EXISTING directory, 2 new files)
├── specialist-routing-guide.md            ~70 lines — when to offer story-architect
└── escalation-triage.md                   ~70 lines — escalation type routing
```

**Total:** ~1,560 lines across 9 new files + 1 KB stub + 1 minimal existing-file append.

---

## Writing Protocol

Applies to all tasks in this plan:

**Path resolution:** `~/.claude/skills/` resolves to `C:\Users\qcivi\.claude\skills\` on this Windows system. All `~/` paths in this plan use this resolution.

**Spec references:** This plan references the spec by section + heading (e.g., "spec Section 4.2, 'The Artifact Schema'"), not by line number. Line numbers are approximate and may shift if the spec is edited.

**File writing rules:**
- **Files over 150 lines:** Break the "Write [file]" step into 2-3 sub-steps grouped by content cluster. Each sub-step targets a logical group of sections.
- **Files under 150 lines:** A single "Write [file]" step is acceptable — these are 5-10 minute writes.
- **Every write step** must cite the specific spec section and heading being transcribed.

**Token efficiency — selective loading:** Reference files use section headings as anchors. SKILL.md's `Read` instructions should specify which section to load, not the entire file. Example: `Read references/artifact-schemas.md#story-manifest-schema` rather than loading all 300 lines. This keeps per-turn context lean.

**Per-step validation:** Every write step is followed by a validation step that defines specific pass/fail criteria (counts, field names, structural checks). The executor must run the validation before proceeding.

**After each chunk:** dispatch plan-document-reviewer subagent with chunk files + spec path. Fix issues and re-dispatch until approved.

**After each chunk:** run a smoke-test verification step (defined per chunk).

**Commit strategy:** Individual commits per task during execution. Squash-merge to main when the full plan is verified. If a chunk smoke test fails, fix forward (don't revert) — the issue is almost always a syntax or reference-path problem.

---

## Chunk 1: Foundation References

Three reference files that establish the field-name contract. No inter-dependencies.

### Task 1: artifact-schemas.md

**Files:**
- Create: `~/.claude/skills/story-architect/references/artifact-schemas.md`

**Done when:** All 11 content sections written, all field names from spec artifact tables present, all 3 provenance states defined, handoff-context.json schema matches spec verbatim, continuity-ledger.md schema included.

This is the single source of truth for every artifact field. Both SKILL.md and friction-patterns.md point here.

**Content sections:**
1. Canonical directory structure (from spec 4.2)
2. `story-manifest.md` full schema — all fields with types, required/optional per genre, provenance tracking (`writer-provided` / `unresolved` / `confirmed`), gate_log format, `[UNRESOLVED: reason]` annotation syntax
3. `structure-map.md` schema — act breakdown fields, turning points, pacing notes
4. `characters/[slug].md` schema — pointer to character-frames.md for genre frame, plus shared fields (role, story_position, voice_anchors, relationships)
5. `cross-character-relationships.md` schema (type, dynamic, status format)
6. `chapters/[nn]-[slug].md` schema (scene_objective, POV, entry/exit beats, tension, character_state_delta, continuity markers, open questions)
7. `handoff-context.json` full JSON schema (from spec Section 2) with versioning rules (MAJOR.MINOR), four load-bearing fields
8. `story-architect-session.json` transient state fields
9. Memory index entry template (from spec 4.3)
10. Field provenance state definitions
11. Escalation payload schemas — both JSON payloads from spec Section 3 Feature 5 (story-architect↔writing-guide)

**Spec sections consumed:** 2 (Persistent Artifacts, Handoff Context), 4.2, 4.3, 4.4, 4.5, 5.4, 5.5, 6.2, 6.5.

- [ ] **Step 1: Create directory structure**

Run: `mkdir -p ~/.claude/skills/story-architect/references`

- [ ] **Step 2: Write artifact-schemas.md — planning artifact schemas (sections 1-6)**

Write sections 1-6: canonical directory structure (spec 4.2 lines 726-745), story-manifest.md full schema (spec 4.2 + 4.4 lines 822-832 for gate_log format + 4.5 lines 863-867 for provenance states), structure-map.md schema, characters/[slug].md schema (with pointer to character-frames.md), cross-character-relationships.md schema (spec 4.2 lines 762-769), continuity-ledger.md schema (spec 6.2 line 1225 — committed facts log, addition workflow: prose-editor flags → writer confirms → write), chapters/[nn]-[slug].md schema.

Include the `[UNRESOLVED: reason]` annotation syntax from spec Section 3 Feature 3 lines 556-557.

- [ ] **Step 3: Write artifact-schemas.md — handoff, session, memory, escalation (sections 7-11)**

Write sections 7-11: handoff-context.json full JSON schema (spec Section 2 lines 306-345) with versioning rules and four load-bearing fields (spec lines 350-354), story-architect-session.json transient state fields (spec 6.2 lines 1213-1214), memory index entry template (spec 4.3 lines 783-798), field provenance state definitions (spec 4.5 lines 863-867), escalation payload schemas — both JSON payloads from spec Section 3 Feature 5 lines 631-670.

- [ ] **Step 4: Verify spec coverage**

Grep artifact-schemas.md for every field name mentioned in the spec's artifact tables (Section 2 lines 85-93, Section 4.2 lines 749-758, Section 6.2 lines 1219-1248). Confirm no field is missing. Specifically verify: `continuity-ledger.md` fields present, `cross-character-relationships.md` fields present, all three provenance states defined.

- [ ] **Step 5: Commit**

```bash
git add ~/.claude/skills/story-architect/references/artifact-schemas.md
git commit -m "feat(story-architect): add artifact-schemas reference — field schemas, handoff JSON, escalation payloads"
```

---

### Task 2: genre-vocabulary.md

**Files:**
- Create: `~/.claude/skills/story-architect/references/genre-vocabulary.md`

Substitution tables for adapting surface language by genre tag. Loaded when friction questions fire or gate prompts render.

**Content sections:**
1. Genre tag definitions — one-line per tag
2. Full substitution table from spec Section 2 lines 174-184: generic → narrative-fiction → memoir → creative-nonfiction → long-form-essay for 7 terms (protagonist, want, wound, arc, inciting event, stakes, theme)
3. Gate question language adaptation rules — triggers identical, only surface nouns change
4. Genre-normative structural differences — fiction misbelief overcome; memoir lie exposed; CNF complicity interrogated; essay governing contradiction may be held permanently (spec line 222)
5. Hybrid genre handling — single primary tag, secondary noted in story-manifest.md

**Spec sections consumed:** 2 (Genre Vocabulary Adaptation, Genre Abstraction Layer), 6.6.

- [ ] **Step 1: Write genre-vocabulary.md**

Transcribe the substitution table exactly from spec Section 2, "Genre Vocabulary Adaptation." Add the genre-normative structural differences from spec Section 2, "Genre Abstraction Layer" (the "Sharpest structural difference" paragraph). Add hybrid genre handling from spec Section 6.6.

- [ ] **Step 2: Validate completeness**

Verify: (1) all 7 generic terms appear (protagonist, want, wound, arc, inciting event, stakes, theme), (2) each has entries for all 4 genre tags, (3) total = 7 × 4 = 28 substitution entries, (4) genre-normative note covers all 4 misbelief-equivalents, (5) hybrid handling rule is present.

- [ ] **Step 3: Commit**

```bash
git add ~/.claude/skills/story-architect/references/genre-vocabulary.md
git commit -m "feat(story-architect): add genre-vocabulary reference — substitution tables for 4 genre tags"
```

---

### Task 3: character-frames.md

**Files:**
- Create: `~/.claude/skills/story-architect/references/character-frames.md`

All four genre-specific character frames in full, with definitions and KB backing.

**Content sections:**
1. Narrative Fiction — want/need/misbelief/arc (backed by `GUIDE.CHARACTER.WANT_NEED_MISBELIEF.md`)
2. Memoir — The Wound / The Reckoning / The Lie the Self Told / The Earned Distance (spec lines 196-202)
3. Creative Nonfiction — The Obsession / The Hidden Claim / The Reporter's Complicity / The Form as Argument (spec lines 206-211)
4. Long-Form Essay — The Animating Question / The Persona / The Governing Contradiction / The Movement of Mind (spec lines 215-220)
5. Cross-genre note on misbelief-equivalent resolution differences (spec line 222)
6. Minimum-viable resolution criteria per frame field, adapted per genre

**Spec sections consumed:** 2 (Genre Abstraction Layer — all four character frame tables).

- [ ] **Step 1: Write character-frames.md**

Transcribe all four character frames from spec Section 2, "Genre Abstraction Layer" (all four character frame tables). Include cross-genre misbelief-resolution note (spec Section 2, "Sharpest structural difference across genres"). Add minimum-viable resolution criteria per frame field. Flag `GUIDE.CRAFT.MEMOIRIST_DISTANCE.md` as not-yet-created for the memoir "Earned Distance" field.

- [ ] **Step 1b: Validate character-frames completeness**

Verify: (1) narrative fiction frame has 4 fields (want/need/misbelief/arc) with definitions, (2) memoir frame has 4 fields (Wound/Reckoning/Lie the Self Told/Earned Distance), (3) CNF frame has 4 fields (Obsession/Hidden Claim/Reporter's Complicity/Form as Argument), (4) essay frame has 4 fields (Animating Question/Persona/Governing Contradiction/Movement of Mind), (5) each field has a definition + fiction analogue + KB backing note, (6) total = 16 frame fields across 4 genres.

- [ ] **Step 2: Create memoir KB stub**

Write `docs/superpowers/writing-kb/writing_mentor_kb/02_leaf_guides/GUIDE.CRAFT.MEMOIRIST_DISTANCE.md`:

```markdown
---
guide_id: GUIDE.CRAFT.MEMOIRIST_DISTANCE
title: "Memoirist Distance — The Earned Distance concept"
doc_type: leaf_guide
status: stub
---

# STUB — Not yet written

Referenced by: memoir character frame ("The Earned Distance" — spec Section 2 line 202)
Needed for: Character Commitment gate, memoir genre tag
KB backing: None yet — requires dedicated research into memoirist epistemological distance

**Workaround:** Until this guide exists, story-architect evaluates the memoir
"Earned Distance" field using the general resolution criteria in
`references/character-frames.md` (clause threshold + specificity threshold)
rather than KB-backed craft evaluation.
```

- [ ] **Step 3: Commit**

```bash
git add ~/.claude/skills/story-architect/references/character-frames.md
git add docs/superpowers/writing-kb/writing_mentor_kb/02_leaf_guides/GUIDE.CRAFT.MEMOIRIST_DISTANCE.md
git commit -m "feat(story-architect): add character-frames reference + memoir KB stub"
```

### Chunk 1: Smoke Test

- [ ] **Verify field-name contract:** Pick 5 field names from the spec's gate map (spec lines 291-296) — `core_tension`, `protagonist_desire`, `act_structure`, `scene_objective`, `thematic_question`. Grep artifact-schemas.md for each. All 5 must appear with type definitions and provenance tracking.

### Chunk 1: Review

- [ ] **Dispatch plan-document-reviewer** with: Chunk 1 files (artifact-schemas.md, genre-vocabulary.md, character-frames.md, GUIDE.CRAFT.MEMOIRIST_DISTANCE.md stub) + spec path. Fix issues and re-dispatch until approved.

---

## Chunk 2: Behavioral References

Two reference files that depend on field names from Chunk 1.

### Task 4: friction-patterns.md

**Files:**
- Create: `~/.claude/skills/story-architect/references/friction-patterns.md`

The complete friction engine specification. This is the behavioral engine SKILL.md invokes by reference.

**Content sections:**
1. **3-pass gap scan** — Pass 1 (field emptiness, placeholder blocklist: `TBD`, `not sure yet`, `something about`, `maybe`, single-word entries); Pass 2 (underdevelopment: clause threshold, specificity threshold, internal completeness); Pass 3 (logical contradiction: want/obstacle, stakes/ending, theme/story, tone/structure mismatches)
2. **Priority + selection rule** — 4-step ranking: severity (contradiction=3, empty=2, underdeveloped=1), category priority (Premise > Character > Structure > Theme), recency suppression (2-turn cooldown), fire exactly one
3. **Friction trigger tables** — all 4 tables from spec Section 2 lines 127-163: Premise (5 rows), Character (5 rows), Structure (4 rows), Theme (4 rows) — using generic vocabulary with genre substitution at render time
4. **Dormant rule** — 5 no-fire conditions (spec lines 166-171)
5. **Answer validation** — [What was heard] / [What's missing] / [Re-framed probe] pattern; 3-attempt re-ask limit with exit phrase (spec lines 516-535)
6. **Hollow compliance detection** — 3 consecutive thin answers under 10 words, advisory naming (spec lines 860-861)
7. **Sprint mode** — entry triggers, listener behavior, exit triggers, post-sprint summary and gap scan resume (spec lines 537-547)
8. **Gap-to-gate-hold rule** — 3-condition escalation (required field + Rigorous/Workshop + Critical/Structural severity); Workshop deferral with 6-field justification (spec lines 266-297)
9. **Gap severity model** — Critical/Structural/Advisory definitions with per-mode behavior table (spec lines 279-286)
10. **Multi-gap answer processing** — acknowledge all resolved fields before next question (spec Section 4.11)

**Spec sections consumed:** 2 (Friction Trigger Model, Gap-to-Gate-Hold Rule), 3 (Features 2, 3), 4.11.

- [ ] **Step 1: Write friction-patterns.md — scan engine + triggers (sections 1-4)**

Write sections 1-4: 3-pass gap scan (spec lines 100-116 — include placeholder blocklist verbatim), priority + selection rule (spec lines 118-123 — 4-step ranking with exact severity scores), all 4 friction trigger tables (spec lines 127-163 — transcribe every row using generic vocabulary), dormant rule (spec lines 166-171 — all 5 no-fire conditions).

- [ ] **Step 2: Write friction-patterns.md — validation + behavioral patterns (sections 5-10)**

Write sections 5-10: answer validation pattern with example (spec lines 516-535 — include the "She wants revenge" example verbatim), hollow compliance detection (spec lines 860-861), sprint mode (spec lines 537-547 — entry triggers, listener behavior, exit triggers), gap-to-gate-hold rule (spec lines 266-297 — include Workshop 6-field deferral), gap severity model (spec lines 279-286 — Critical/Structural/Advisory table), multi-gap answer processing (spec Section 4.11 lines 986-997 — include the "resolved three things" acknowledgment template).

- [ ] **Step 3: Verify all trigger conditions from spec are present**

Cross-check against spec Section 2 trigger tables — count rows: Premise (5), Character (5), Structure (4), Theme (4) = 18 total. Verify all 18 appear. Verify all 5 dormant conditions appear. Verify the 4 contradiction types from Pass 3 appear.

- [ ] **Step 4: Commit**

```bash
git add ~/.claude/skills/story-architect/references/friction-patterns.md
git commit -m "feat(story-architect): add friction-patterns reference — gap scan, triggers, validation, sprint mode"
```

---

### Task 5: kb-gate-map.md

**Files:**
- Create: `~/.claude/skills/story-architect/references/kb-gate-map.md`

Which KB files load at which gates, required fields per gate x genre, gap severity assignments.

**Content sections:**
1. **Loading rule** — KB files load on gate activation, unload on gate close (spec Section 4.7)
2. **Gate-to-KB mapping** (from spec Section 4.7 lines 898-903):
   - Premise Lock: `GUIDE.PREMISE.VIABILITY.md`, `CHK.PREMISE.VIABILITY.md`
   - Character Commitment: `GUIDE.CHARACTER.WANT_NEED_MISBELIEF.md`, `GUIDE.CHARACTER.AGENCY_AND_DECISION_PRESSURE.md`
   - Structure Approval: `GUIDE.STRUCTURE.WEILAND_BEAT_MAP.md`, `GUIDE.STRUCTURE.TWO_HALVES_MAJOR_BEATS.md`, `GUIDE.STRUCTURE.MIDPOINT_SHIFT.md`
   - Chapter Direction: `CHK.SCENE.PURPOSE.md`, `MAP.CONTINUITY.LEDGER_TEMPLATE.md`
3. **Required field sets per gate x genre** — the full Gate Map from spec lines 291-296 (Premise Lock, Character Commitment, Structure Approval, Chapter Direction with required fields per genre)
4. **Gap severity assignments** — which fields are Critical, Structural, Advisory (static per gate x genre, from spec lines 279-286)
5. **Context budget management** — tiered loading strategy table from spec Section 4.6 lines 878-887, 60% capacity warning
6. **KB coverage gaps** — note that `GUIDE.CRAFT.MEMOIRIST_DISTANCE.md` is referenced but not yet created; note fiction-primary KB depth vs non-fiction thinness
7. **Token budget estimates per gate** — estimated tokens for each gate's load set (KB files + relevant reference file sections + always-loaded story-manifest.md). Include the 60% capacity threshold calculation basis.

**Spec sections consumed:** 2 (Gap-to-Gate-Hold Rule gate map), 4.6, 4.7, 6.3, 7.3.

- [ ] **Step 1: Write kb-gate-map.md**

Transcribe gate-to-KB mapping and required field tables. Add KB path prefix note (all relative to `docs/superpowers/writing-kb/writing_mentor_kb/`). Document the memoir KB gap.

- [ ] **Step 2: Verify all 8 KB files referenced exist in the KB directory**

Run: `ls docs/superpowers/writing-kb/writing_mentor_kb/02_leaf_guides/GUIDE.PREMISE.VIABILITY.md` (and all 7 others)

- [ ] **Step 3: Commit**

```bash
git add ~/.claude/skills/story-architect/references/kb-gate-map.md
git commit -m "feat(story-architect): add kb-gate-map reference — gate-to-KB mapping, required fields, severity"
```

### Chunk 2: Smoke Test

- [ ] **Trace one friction scenario end-to-end:** Scenario: `core_conflict` is empty for a `narrative-fiction` project in Rigorous mode. Verify in friction-patterns.md: (1) Pass 1 detects emptiness, (2) severity = 2 (empty), (3) category priority = Premise (highest), (4) friction trigger table row fires: "What force or situation is actively working against what your protagonist wants most?", (5) genre-vocabulary.md confirms "protagonist" is the correct generic term for narrative-fiction, (6) kb-gate-map.md confirms `core_conflict` is a required field at Premise Lock for narrative-fiction with Critical severity.

### Chunk 2: Review

- [ ] **Dispatch plan-document-reviewer** with: Chunk 2 files (friction-patterns.md, kb-gate-map.md) + spec path. Fix issues and re-dispatch until approved.

---

## Chunk 3: Main Skill File

### Task 6: SKILL.md

**Files:**
- Create: `~/.claude/skills/story-architect/SKILL.md`

**Done when:** ≤500 lines, all 14 sections from budget present, every `Read references/` instruction resolves to existing file+section, skill loads on `/story-architect` invocation, intake flow works end-to-end.

The main skill file. ~480 lines, hard cap 500. Orchestrates by referencing all 5 reference files. Contains behavioral instructions; references/ provides data tables.

**Key authoring principle:** SKILL.md tells Claude WHEN to do something and WHERE to find details. Every section that could expand past 10-15 lines of detail contains a `Read references/[file].md` instruction with a clear load condition.

**Section budget:**

| Section | ~Lines | Content |
|---|---|---|
| Frontmatter | 15 | name, pushy description (see below), entry_modes, kb_root |
| Hard Boundaries | 20 | Anti-ghostwriting 5 rules with reasoning (why passive agreement doesn't count, why annotations persist), one-question-per-turn with reasoning, Write Notice/Receipt requirement. Distribute constraint reasoning here rather than in a separate guardrails section. |
| Entry Points | 40 | Direct invocation (`/story-architect [args]`), skill-to-skill API, parameter schema |
| Session Start / Resume | 60 | 5-step decision tree (spec lines 362-366), resume summary template (spec lines 372-395), stale artifact tiers (4), mid-gate resume, mode change on resume |
| Session Modes | 50 | Exploratory/Rigorous/Workshop in **table format** (see below), key distinction, switching rules |
| Gate System | 80 | Sequence (Intake → Premise Lock → Character Commitment → Structure Approval → Chapter Direction → Handoff), interaction flow (spec lines 480-500), genre tag at intake, pointers to kb-gate-map.md. Include gate-hold reasoning inline. |
| Friction Engine | 30 | 3-pass scan overview, pointer to friction-patterns.md, pointer to genre-vocabulary.md |
| Artifact Pipeline | 40 | Stub→full progression (spec lines 586-615), Write Notice/Receipt templates, pointer to artifact-schemas.md, gap annotation rules with reasoning (spec Section 3 Feature 3) |
| Anti-Drift Controls | 35 | Annotation persistence with reasoning (why silent removal is dangerous), gate commitment, coherence alert template (spec lines 920-931), genre tag re-evaluation |
| Integration | 30 | Two routing directions, escalation payloads (pointer to artifact-schemas.md), return bridge pattern (spec lines 646-655) |
| Writer Commands | 15 | `plan summary`, `gap report` — conversation-only |
| Diagnostic Intervention | 20 | Trigger conditions, response format (spec lines 954-976) |
| Context Budget | 15 | Tiered loading overview, pointer to kb-gate-map.md, 60% warning |
| Session State | 20 | story-architect-session.json fields, memory index lifecycle |
| Special Protocols | 10 | Real-person disclosure for memoir (spec line 1005), handoff_notes constraint |

**Note: No separate Guardrails section.** Constraint reasoning is distributed into Hard Boundaries, Gate System, and Anti-Drift Controls where each constraint operates. This follows the skill-creator principle of "explain the why" rather than extracting rules into a rigid list.

**Frontmatter description (draft — optimize post-build):**

```yaml
description: >
  Structured story-planning coach for writers building a story from scratch.
  Use this skill whenever a writer wants to develop a story idea, plan a novel
  or memoir, build character arcs, map story structure, stress-test a premise,
  organize scattered notes into a framework, or move from a vague concept to
  a chapter-by-chapter plan. Also use when a writer returns to continue an
  in-progress story project, or when writing-guide detects a structural
  planning need that exceeds its scope. Handles narrative fiction, memoir,
  creative nonfiction, and long-form essay. Works through planning gates with
  targeted friction questions — never writes prose or generates content.
```

**Session modes table format (fits ~40 lines, preserves all behavioral detail):**

```markdown
| Aspect | Exploratory | Rigorous | Workshop |
|---|---|---|---|
| Opening | One orienting question | Status check → active gates | Formal frame + evaluation criteria |
| Question style | Curious, lateral, non-evaluative | Direct, sequential, builds toward assessment | Socratic, adversarial (productively) |
| Gap handling | Noted lightly, optional | Named, tracked, significant gaps block gates | Treated as claims requiring response |
| Stuck response | "I don't know" is useful; explore/park/constrain | Diagnose type, targeted prompts | Characterize stuckness, articulate resolution |
| Positive feedback | Reflective only | Brief, specific, proportionate | High bar, structural, earned |
| Escalation | Suggests Rigorous once if gaps costly | Suggests Exploratory/Workshop if appropriate | Suggests Rigorous if avoidance detected |
| Tone | Spacious, unhurried, lateral | Precise, structured, purposeful | Rigorous, demanding, collegial |
```

- [ ] **Step 1: Write SKILL.md frontmatter and Hard Boundaries**

Frontmatter: name, the pushy description from the section budget above (adapt as needed), entry_modes (direct + skill_api), kb_root pointing to writing_mentor_kb. Hard Boundaries: transcribe the 5 anti-ghostwriting rules WITH reasoning from spec Section 4.5, "Anti-Ghostwriting Enforcement" (why passive agreement doesn't count, why "yes" to a skill-generated suggestion isn't writer-provided). Add one-question-per-turn rule with reasoning from spec Section 3, Feature 2, "Why one question per turn." Add Write Notice/Receipt requirement with reasoning from spec Section 3, Feature 4, "What the writer sees at each write."

- [ ] **Step 2: Write Entry Points section**

Direct invocation commands, skill-to-skill parameter schema. Model after writing-guide's entry point format.

- [ ] **Step 3: Write Session Start / Resume section**

5-step decision tree, resume summary template (use the exact template from spec lines 372-395), stale artifact tiers, mid-gate resume, mode change rules.

- [ ] **Step 4: Write Session Modes section**

Use the **table format** from the section budget above (7-row table: opening, question style, gap handling, stuck response, positive feedback, escalation, tone). Add the key distinction: Rigorous asks "is this resolved?", Workshop asks "does this hold under contradiction?" Add switching rules: upgrade shows what's added + confirms, downgrade shows what's removed + warns, applies prospectively only, recorded in mode_history array (spec lines 408-409).

- [ ] **Step 5: Write Gate System section**

Gate sequence diagram, interaction flow, gate overviews. Every gate section includes: `Read references/kb-gate-map.md` load condition. Genre tag at intake rules (changeable before Premise Lock; after requires gate revision).

- [ ] **Step 6: Write Friction Engine section**

Overview of 3-pass scan with `Read references/friction-patterns.md` load condition. Overview of genre vocabulary substitution with `Read references/genre-vocabulary.md` load condition.

- [ ] **Step 7: Write Artifact Pipeline section**

Stub→full progression. Write Notice template: "Writing to [path]: [what changed]". Write Receipt template: "Saved: [path]". Pointer to artifact-schemas.md for field definitions. Gap annotation format: `[UNRESOLVED: reason]`.

- [ ] **Step 8: Write Anti-Drift Controls + Integration + Writer Commands**

Anti-Drift Controls (~35 lines): 4 mechanisms from spec Section 4.8 — annotation persistence with reasoning (spec Section 3 Feature 3 lines 559-573), gate clearance as commitment, coherence alert template (spec lines 922-931), genre tag change re-evaluation. Integration (~30 lines): two routing directions from spec Section 3 Feature 5, escalation payloads (pointer to artifact-schemas.md), return bridge pattern (spec lines 646-655). Writer Commands (~15 lines): `plan summary` and `gap report` — conversation-only, no artifact writes.

- [ ] **Step 9: Write Diagnostic + Context Budget + Session State + Special Protocols**

Diagnostic Intervention (~20 lines): trigger conditions, response format from spec lines 954-976. Context Budget (~15 lines): tiered loading overview, pointer to kb-gate-map.md, 60% capacity warning. Session State (~20 lines): story-architect-session.json fields (spec 6.2 line 1214), memory index lifecycle (spec 4.3 lines 809-813). Special Protocols (~10 lines): real-person disclosure for memoir (spec line 1005), handoff_notes factual-only constraint (spec line 1008). No separate Guardrails section — reasoning is already distributed.

- [ ] **Step 10: Line count check**

Run: `wc -l ~/.claude/skills/story-architect/SKILL.md`
Expected: ≤500 lines. If over, move detail to appropriate reference file (session modes table → references/session-modes.md is the first candidate).

- [ ] **Step 11: Reference chain verification**

Grep SKILL.md for every `Read references/` instruction. Verify each target file and section exists in the references/ directory created in Chunks 1-2.

- [ ] **Step 12: Commit**

```bash
git add ~/.claude/skills/story-architect/SKILL.md
git commit -m "feat(story-architect): add main SKILL.md — gate system, friction engine, session management"
```

### Chunk 3: Smoke Test

- [ ] **Invoke `/story-architect` and verify intake flow:** Invoke the skill with no arguments. Verify it: (1) loads without error, (2) asks for genre tag selection, (3) presents session mode selection with Exploratory as default, (4) after receiving a premise fragment, runs gap scan and fires one friction question on the highest-priority gap. If the skill fails to load, check frontmatter syntax and reference paths.

### Chunk 3: Review

- [ ] **Dispatch plan-document-reviewer** with: Chunk 3 file (SKILL.md) + spec path + all 5 reference files for cross-reference verification. Fix issues and re-dispatch until approved.

---

## Chunk 4: writing-guide Extensions

Two thin reference files extending writing-guide, plus a minimal SKILL.md append.

### Task 7: specialist-routing-guide.md

**Files:**
- Create: `~/.claude/skills/writing-guide/references/specialist-routing-guide.md`

Runtime reference for writing-guide to determine when to offer story-architect.

**Content sections:**
1. Routing decision tree — structural planning need → offer story-architect; prose consistency → offer prose-editor (noted as future/unavailable); general craft → handle in writing-guide
2. Entry triggers for story-architect (3 triggers from spec Section 5.2): explicit invocation, toll gate escalation, session resume
3. Entry triggers for prose-editor (future, documented for completeness)
4. Offer framing — transitions offered not forced, writer confirms
5. What writing-guide does NOT do — no story structure guidance, no prose review
6. Session state detection — how writing-guide checks for active story-architect session

**Spec sections consumed:** 5.1, 5.2, 5.6.

- [ ] **Step 1: Write specialist-routing-guide.md**

Content sections from spec: routing decision tree (Section 5.1, "Architectural Model"), three entry triggers for story-architect (Section 5.2, "Entry Points"), offer framing (Section 5.7, principle 5: "Transitions are offered, not triggered"), session state detection via `story-architect-session.json` (Section 6.2).

- [ ] **Step 2: Validate routing completeness**

Verify: (1) all 3 story-architect entry triggers present (explicit invocation, toll gate escalation, session resume), (2) prose-editor triggers documented as future, (3) routing decision tree has exactly 3 terminal paths (story-architect, prose-editor, handle-in-writing-guide), (4) offer framing includes writer confirmation requirement.

- [ ] **Step 3: Commit**

```bash
git add ~/.claude/skills/writing-guide/references/specialist-routing-guide.md
git commit -m "feat(writing-guide): add specialist-routing-guide — when to offer story-architect"
```

---

### Task 8: escalation-triage.md

**Files:**
- Create: `~/.claude/skills/writing-guide/references/escalation-triage.md`

How writing-guide interprets and routes escalation payloads from specialist modules.

**Content sections:**
1. Escalation type table from spec Section 5.5 lines 1110-1117: structural-gap, voice-drift, continuity-contradiction, gate-hold, diagnostic, ghostwriting-risk — with source, routes-to, severity, writer action required
2. Payload interpretation rules
3. Routing rules — structural rework returns to source
4. Writer-facing communication — plan-vs-prose decision framing (spec Section 5.3 lines 1076-1081)
5. Return loops — non-linear re-entry patterns

**Spec sections consumed:** 5.3, 5.4, 5.5.

- [ ] **Step 1: Write escalation-triage.md**

Content sections from spec: escalation type table (Section 5.5, "Escalation Types and Routing" — all 6 types), payload interpretation rules (Section 5.4, "Inter-Module Handoffs"), plan-vs-prose decision framing (Section 5.3, the "Structural conflict detected" example), return loops (Section 5.3, "Return loops — non-linear re-entry").

- [ ] **Step 2: Validate escalation completeness**

Verify: (1) all 6 escalation types present (structural-gap, voice-drift, continuity-contradiction, gate-hold, diagnostic, ghostwriting-risk), (2) each type has source, routes-to, severity, and writer-action-required, (3) ghostwriting-risk is marked as non-routable (enforced locally), (4) plan-vs-prose decision template includes all 3 writer options (revise plan, revise prose, defer).

- [ ] **Step 3: Commit**

```bash
git add ~/.claude/skills/writing-guide/references/escalation-triage.md
git commit -m "feat(writing-guide): add escalation-triage — escalation type routing for specialist modules"
```

---

### Task 9: writing-guide SKILL.md — load condition append

**Files:**
- Modify: `~/.claude/skills/writing-guide/SKILL.md` (append only — 4 lines at end of file)

**Decision gate:** The spec says "existing, ~625 lines, do not modify" for writing-guide's SKILL.md. However, the two new reference files (specialist-routing-guide.md, escalation-triage.md) cannot be loaded at runtime unless SKILL.md contains `Read` instructions for them. **Ask the user before proceeding.** If declined, document the integration gap and skip this task.

- [ ] **Step 1: Ask user for approval to append to writing-guide SKILL.md**

Present: "The spec says not to modify writing-guide's SKILL.md, but the new routing references can't load without it. Append these 4 lines at the end? Y/N"

- [ ] **Step 2: Append load conditions (if approved)**

Append to end of `~/.claude/skills/writing-guide/SKILL.md`:

```markdown

## Specialist Routing (loaded on demand)

When a writer's need exceeds general craft guidance, check: `Read references/specialist-routing-guide.md`
When processing an escalation payload from a specialist module: `Read references/escalation-triage.md`
```

- [ ] **Step 3: Commit**

```bash
git add ~/.claude/skills/writing-guide/SKILL.md
git add ~/.claude/skills/writing-guide/references/specialist-routing-guide.md
git add ~/.claude/skills/writing-guide/references/escalation-triage.md
git commit -m "feat(writing-guide): add specialist routing + escalation triage references"
```

### Chunk 4: Smoke Test

- [ ] **Verify writing-guide loads new references:** If Task 9 was approved, invoke `/writing-guide` and present a structural planning need (e.g., "I have a story idea but no premise"). Verify writing-guide: (1) loads without error, (2) reads specialist-routing-guide.md, (3) offers to invoke story-architect. If Task 9 was skipped, verify the reference files exist in the correct directory and document the integration gap.

### Chunk 4: Review

- [ ] **Dispatch plan-document-reviewer** with: Chunk 4 files (specialist-routing-guide.md, escalation-triage.md, writing-guide SKILL.md diff if applicable) + spec path. Fix issues and re-dispatch until approved.

---

## Verification

After all 8 files are written and committed:

- [ ] **V1: Cross-file field name consistency** — grep all 8 files for field names from artifact-schemas.md; verify consistent naming
- [ ] **V2: Reference chain integrity** — every `Read references/X.md` in SKILL.md resolves to an existing file and section
- [ ] **V3: Gate required-field consistency** — compare kb-gate-map.md required fields with friction-patterns.md trigger conditions and SKILL.md gate descriptions
- [ ] **V4: Escalation payload consistency** — compare artifact-schemas.md payloads with specialist-routing-guide.md triggers and escalation-triage.md types
- [ ] **V5: Genre vocabulary completeness** — every friction question in friction-patterns.md uses generic vocabulary with a substitution entry in genre-vocabulary.md
- [ ] **V6: Line count compliance** — SKILL.md ≤500 lines
- [ ] **V7: Structured test scenarios** — run these 3 scenarios and verify expected behavior:

**Scenario 1 — New fiction project:**
Input: `/story-architect` then "I have an idea for a novel about a journalist who discovers her mother was a Cold War spy. I don't know where to start."
Expected: (1) Skill loads, (2) asks for genre tag — writer selects `narrative-fiction`, (3) presents session mode with Exploratory default, (4) creates story-manifest.md stub at intake, (5) runs gap scan, fires one friction question targeting `core_conflict` (highest priority empty field).

**Scenario 2 — Memoir resume:**
Setup: Create a story-manifest.md stub in `docs/writing/test-memoir/` with `genre_tag: memoir`, `premise_lock: cleared: 2026-03-28`, `character_commitment: in progress`. Create a memory index entry pointing to it.
Input: `/story-architect`
Expected: (1) 5-step resume flow activates, (2) reads story-manifest.md (not memory), (3) presents resume summary template populated from artifact, (4) continues at Character Commitment gate, (5) loads memoir character frame vocabulary ("The Wound" / "The Reckoning" etc.).

**Scenario 3 — Craft question routing:**
Setup: Active story-architect session mid-Premise Lock gate.
Input: "What makes a good midpoint shift?"
Expected: (1) story-architect detects craft question, (2) routes to writing-guide with `kb_hint: GUIDE.STRUCTURE.MIDPOINT_SHIFT.md`, (3) writing-guide response appears inline, (4) story-architect resumes with bridge: "Back to planning. We were working on [field]. Ready to continue?"
