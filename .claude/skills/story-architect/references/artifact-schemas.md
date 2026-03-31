# Artifact Schemas

> Single source of truth for every artifact field in the story-architect skill.
> SKILL.md and friction-patterns.md point here for field definitions.

---

## Table of Contents

1. [Canonical Directory Structure](#canonical-directory-structure)
2. [Field Provenance States](#field-provenance-states)
3. [story-manifest.md Schema](#story-manifestmd-schema)
4. [structure-map.md Schema](#structure-mapmd-schema)
5. [characters/[slug].md Schema](#charactersslugmd-schema)
6. [cross-character-relationships.md Schema](#cross-character-relationshipsmd-schema)
7. [continuity-ledger.md Schema](#continuity-ledgermd-schema)
8. [chapters/[nn]-[slug].md Schema](#chaptersnnn-slugmd-schema)
9. [handoff-context.json Schema](#handoff-contextjson-schema)
10. [story-architect-session.json Schema](#story-architect-sessionjson-schema)
11. [Memory Index Entry Template](#memory-index-entry-template)
12. [Escalation Payload Schemas](#escalation-payload-schemas)

---

## Canonical Directory Structure

```
docs/writing/[project-slug]/
├── story-manifest.md
├── structure-map.md
├── cross-character-relationships.md
├── [project-slug]-kb-patch.md          (optional)
├── continuity-ledger.md
├── characters/
│   └── [character-slug].md
├── chapters/
│   ├── [nn]-[slug].md
│   └── [nn]-[slug]-review.md           (prose-editor output)
├── handoff/
│   ├── handoff-context-v1.json
│   └── handoff-context-v2.json
└── checkpoints/
    └── [YYYY-MM-DD]-manifest.md
```

All paths are relative to the repository root. The `[project-slug]` segment is a kebab-case identifier derived from the project title at intake.

---

## Field Provenance States

Every story field in the manifest and downstream artifacts carries one of three provenance states. There is NO "skill-generated" state -- content authored by the skill is never written to artifacts.

| State | Definition | Annotation |
|---|---|---|
| `writer-provided` | Content originated from writer input and passes minimum-viable resolution. | No annotation needed. |
| `unresolved` | Field is empty, thin, or unanswered. | `[UNRESOLVED: reason]` -- persists until field passes minimum-viable resolution. |
| `confirmed` | Writer-provided AND explicitly confirmed during gate clearance; logged in `gate_log`. Assigned at gate passage as a batch operation. Cannot be inferred from conversational context. | No annotation; confirmation is recorded in the gate_log. |

### Annotation Syntax

Inline annotation for unresolved fields:

```
core_tension: "[UNRESOLVED: writer indicated conflict but hasn't articulated opposing forces]"
```

The `[UNRESOLVED: reason]` marker persists in the field value until the writer provides content that passes minimum-viable resolution.

---

## story-manifest.md Schema

The story manifest is the root planning artifact. Field requirements vary by genre -- see the `required` annotations.

### Project Metadata

| Field | Type | Required | Notes |
|---|---|---|---|
| `project` | string | all genres | Kebab-case slug. |
| `title` | string | all genres | Working title. |
| `genre_tag` | enum | all genres | `narrative-fiction` \| `memoir` \| `creative-nonfiction` \| `long-form-essay` |
| `session_mode` | enum | all genres | `exploratory` \| `rigorous` \| `workshop` |
| `created_date` | ISO date | all genres | Date of project creation. |

### Core Story Fields

| Field | Type | Required | Notes |
|---|---|---|---|
| `logline` | string | fiction, memoir | One-sentence story summary. |
| `core_tension` | string | fiction, memoir, creative-nonfiction | The central opposing forces. |
| `protagonist_stake` | string | fiction, memoir | What the protagonist stands to lose. |
| `thematic_question` | string | fiction, memoir, creative-nonfiction | The question the story interrogates. |
| `inciting_premise` | string | fiction, memoir | The event or condition that launches the story. |

### Voice and Tone Fields

| Field | Type | Required | Notes |
|---|---|---|---|
| `tone_markers` | string[] | all genres | Writer-selected descriptors of tonal quality. |
| `narrative_voice` | string | fiction, memoir | POV, tense, distance, register. |
| `argument_claim` | string | long-form-essay only | The central claim or thesis for essay genre. |

### Gate Log

The `gate_log` section tracks progression through planning gates. Each gate entry uses one of the following status values:

| Status | Meaning |
|---|---|
| `not cleared` | Gate has not been attempted. |
| `in progress` | Gate work is underway. |
| `cleared: [date]` | Gate passed on the given ISO date. |
| `revised: [date] (originally cleared: [date])` | Gate was revisited and re-cleared; original clearance date preserved. |

Example:

```yaml
gate_log:
  premise:
    status: "cleared: 2026-03-15"
  structure:
    status: "in progress"
  characters:
    status: "not cleared"
  world_or_context:
    status: "not cleared"
  scene_sequence:
    status: "not cleared"
  theme_statement:
    status: "not cleared"
  voice_profile:
    status: "not cleared"
```

### Provenance Tracking in the Manifest

Every story field in the manifest carries a provenance state (`writer-provided`, `unresolved`, or `confirmed`) as defined in [Field Provenance States](#field-provenance-states). Fields that are `unresolved` embed the `[UNRESOLVED: reason]` annotation directly in their value. Fields that are `confirmed` have their confirmation recorded in the `gate_log` at gate passage as a batch operation.

---

## structure-map.md Schema

The structure map captures the arc-level design of the project.

| Field | Type | Required | Notes |
|---|---|---|---|
| `act_structure` | string | all genres | E.g. "three-act", "four-act", "episodic". |
| `opening_condition` | string | all genres | State of the world/protagonist at story open. |
| `ending_type` | string | all genres | E.g. "resolved", "ambiguous", "circular". |
| `midpoint_shift` | string | fiction, memoir | The reversal or escalation at the structural midpoint. |
| `key_turning_points` | string[] | all genres | Ordered list of major structural beats. |
| `pacing_notes` | string | all genres | Writer's notes on rhythm, tempo, compression. |
| `open_structure_questions` | string[] | all genres | Unresolved structural decisions. |

---

## characters/[slug].md Schema

Character profiles use genre-specific frames. See `character-frames.md` for the full frame definitions per genre. The fields below are shared across all genres.

### Shared Character Fields

| Field | Type | Required | Notes |
|---|---|---|---|
| `name` | string | all genres | Character's name. |
| `role` | string | all genres | `protagonist` \| `antagonist` \| `supporting` \| `narrator` |
| `story_position` | string | all genres | Functional role in the plot or argument. |
| `voice_anchors` | string[] | all genres | Collected at Character Commitment gate. Distinctive speech patterns, diction markers, rhythmic tendencies. |
| `key_relationships` | array of objects | all genres | Each entry: `{ character: string, type: string, dynamic: string }` |
| `unresolved_questions` | string[] | all genres | Open questions about the character. |

### Genre-Specific Frames

Genre-specific character frame fields are defined in `character-frames.md`. The character file includes the shared fields above plus whichever genre frame applies, determined by the `genre_tag` in the story manifest.

### Gate-Required Character Fields

These field names are used in the gate map for Character Commitment evaluation. They map to genre-specific frame concepts:

| Gate Field | narrative-fiction maps to | memoir maps to | creative-nonfiction maps to | long-form-essay maps to |
|---|---|---|---|---|
| `protagonist_desire` | Want | The Wound | The Obsession | The Animating Question |
| `protagonist_wound` | Misbelief | The Lie the Self Told | The Reporter's Complicity | The Governing Contradiction |
| `antagonist_function` | Antagonist's role in blocking the want | N/A (advisory) | N/A (advisory) | N/A (advisory) |
| `character_arc_type` | Arc | The Earned Distance | The Form as Argument | The Movement of Mind |
| `narrator_stance` | N/A | Narrator's relationship to past self | N/A | Speaker's rhetorical position |
| `key_relationships_mapped` | N/A | Key relationships confirmed | N/A | N/A |
| `speaker_ethos` | N/A | N/A | N/A | Constructed credibility of the essayist |

---

## cross-character-relationships.md Schema

This artifact maps every significant relationship between characters. Each entry follows this format:

```
[Character A] -- [Character B]:
  type: [relationship type]
  dynamic: [writer-provided description]
  status: confirmed | [UNRESOLVED: dynamic not yet articulated]
```

### Field Definitions

| Field | Type | Notes |
|---|---|---|
| Character A | string | Name of the first character. |
| Character B | string | Name of the second character. |
| `type` | string | Category of relationship (e.g. "mentor-student", "rivals", "siblings"). |
| `dynamic` | string | Writer-provided description of the relationship's nature and tension. |
| `status` | string | `confirmed` or `[UNRESOLVED: reason]` per provenance rules. |

Relationships are listed once per pair. Directionality, where it matters, is captured in the `dynamic` description.

---

## continuity-ledger.md Schema

The continuity ledger is a running log of committed facts that prose-editor and story-architect use to maintain consistency.

### Entry Fields

| Field | Type | Notes |
|---|---|---|
| `fact_id` | string | Unique identifier for the fact (e.g. `FACT-001`). |
| `fact` | string | The committed fact statement. |
| `established_in` | string | Chapter reference where the fact first appears (e.g. `ch-03-the-arrival`). |
| `confirmed_by` | string | `writer` \| `prose-editor` |
| `status` | string | `active` \| `superseded` |

### Example Entry

```markdown
| fact_id  | fact                                      | established_in     | confirmed_by | status |
|----------|-------------------------------------------|--------------------|--------------|--------|
| FACT-001 | Elena has a scar on her left forearm.      | ch-02-the-crossing | writer       | active |
| FACT-002 | The tavern is on Maple Street.             | ch-01-opening      | prose-editor | active |
| FACT-003 | Elena's eyes are green.                   | ch-01-opening      | writer       | superseded |
```

### Addition Workflow

1. Prose-editor flags candidate facts during drafting or review.
2. Writer confirms or rejects each candidate.
3. Confirmed facts are written to the ledger with `confirmed_by: writer`.
4. Facts flagged by prose-editor but not yet confirmed carry `confirmed_by: prose-editor` and are treated as provisional.
5. Superseded facts retain their entry with `status: superseded` for audit trail.

---

## chapters/[nn]-[slug].md Schema

Chapter planning documents capture scene-level intent and state transitions.

| Field | Type | Notes |
|---|---|---|
| `chapter_number` | int | Sequential chapter number. |
| `slug` | string | Kebab-case chapter identifier. |
| `pov` | string | Point-of-view character for this chapter. |
| `scene_objective` | string | What the scene must accomplish. |
| `entry_state` | string | Emotional/situational state at chapter open. |
| `exit_state` | string | Emotional/situational state at chapter close. |
| `tension_raised` | string | What new tension or question the chapter introduces. |
| `character_state_delta` | string | How characters change through this chapter. |
| `continuity_markers` | string[] | Facts from the continuity ledger referenced or established. |
| `open_questions` | string[] | Unresolved questions entering or exiting this chapter. |

Review files (`[nn]-[slug]-review.md`) are generated by prose-editor and live alongside the chapter plan.

---

## handoff-context.json Schema

The handoff context is the machine-readable bridge between story-architect and prose-editor. This schema must be treated as a contract.

### Versioning

- Format: `MAJOR.MINOR`
- MAJOR bump: breaking changes (field removals, type changes, structural reorganization).
- MINOR bump: backward-compatible additions.
- `extended` buckets at each level are the safe expansion surface for minor additions.

### Full Schema

```json
{
  "schema_version": "1.0",
  "handoff_id": "uuid",
  "created_at": "ISO8601",
  "project": {
    "id": "slug",
    "title": "string",
    "genre_tag": "narrative-fiction | memoir | creative-nonfiction | long-form-essay",
    "base_path": "docs/writing/[project-id]",
    "author_note": "optional, max 300 chars"
  },
  "gates": {
    "premise": { "status": "locked|provisional|skipped|not_started", "locked_at": "", "note": "" },
    "structure": { "status": "locked|provisional|skipped|not_started", "locked_at": "", "note": "" },
    "characters": { "status": "locked|provisional|skipped|not_started", "locked_at": "", "note": "" },
    "world_or_context": { "status": "locked|provisional|skipped|not_started", "locked_at": "", "note": "" },
    "scene_sequence": { "status": "locked|provisional|skipped|not_started", "locked_at": "", "note": "" },
    "theme_statement": { "status": "locked|provisional|skipped|not_started", "locked_at": "", "note": "" },
    "voice_profile": { "status": "locked|provisional|skipped|not_started", "locked_at": "", "note": "" },
    "extended": {}
  },
  // NOTE: Gate key mapping — the handoff JSON uses 7 granular keys (from the spec's
  // prose-editor intake contract) while SKILL.md uses 6 planning gates. The mapping:
  //   "premise"          ← Premise Lock gate
  //   "characters"       ← Character Commitment gate
  //   "structure"        ← Structure Approval gate
  //   "scene_sequence"   ← Chapter Direction gate
  //   "world_or_context" ← populated during intake/premise (no dedicated gate)
  //   "theme_statement"  ← confirmed at Premise Lock (thematic_question field)
  //   "voice_profile"    ← collected at Character Commitment (voice_anchors field)
  // story-architect updates these keys when the corresponding gate clears.
  "artifacts": {
    "premise_doc": { "path": "", "format": "", "description": "", "last_modified": "" },
    "outline": { "path": "", "format": "", "description": "", "last_modified": "" },
    "character_profiles": [],
    "voice_doc": { "path": "", "format": "", "description": "", "last_modified": "" },
    "drafts_dir": "drafts/",
    "extended": {}
  },
  "open_questions": [
    { "id": "", "question": "", "priority": "high|medium|low", "affects": [], "writer_note": "" }
  ],
  "session_mode": {
    "default_mode": "drafting|revision|scene-by-scene|free-write",
    "checkpoint_frequency": "per_scene|per_chapter|per_session|on_demand",
    "gate_surfacing": "silent|on_conflict|always"
  },
  "handoff_notes": "skill-generated summary, max 500 chars"
}
```

### Load-Bearing Fields

Prose-editor must treat these four areas as authoritative:

1. **`project.genre_tag`** -- editorial lens selector. Determines which editorial standards, character frames, and craft guidance apply.
2. **`gates`** -- locked gates are hard constraints. Prose-editor must not contradict or reopen locked gates without explicit writer instruction.
3. **`artifacts`** -- file index. Prose-editor loads content from the paths listed here, never from inline data in the handoff block.
4. **`open_questions` + `gate_surfacing`** -- govern when prose-editor interrupts the writer. When `gate_surfacing` is `silent`, open questions are not surfaced during drafting. When `on_conflict`, only questions relevant to a detected inconsistency are raised. When `always`, all open questions are surfaced at session start.

### Gate Status Values (Handoff Context)

| Status | Meaning |
|---|---|
| `locked` | Gate passed and committed; hard constraint for prose-editor. |
| `provisional` | Gate addressed but not fully locked; prose-editor may surface conflicts. |
| `skipped` | Writer explicitly chose to skip this gate. |
| `not_started` | Gate has not been attempted. |

Note: these status values differ from the gate_log statuses in story-manifest.md. The manifest tracks the planning process; the handoff context communicates constraints to prose-editor.

---

## story-architect-session.json Schema

Transient session state. Not committed to git. Used to maintain context within a single story-architect interaction.

| Field | Type | Notes |
|---|---|---|
| `active_gate` | string | The gate currently being worked on. |
| `project_slug` | string | Current project identifier. |
| `session_mode` | string | `exploratory` \| `rigorous` \| `workshop` |
| `context_budget_state` | object | Tracks token budget allocation across loaded artifacts. |
| `last_friction_field` | string | The field that last triggered a friction pattern. |
| `last_friction_turn` | int | Turn number when friction was last applied. |
| `mode_history` | array of objects | Each entry: `{ from: string, to: string, turn: int }`. Records mode switches for session continuity. |
| `last_session_status` | string | `mid_gate` \| `between_gates` \| `project_complete` \| `intake_incomplete`. Evaluated at resume to determine session start behavior. |

This file is ephemeral. It is regenerated at session start and discarded at session end.

---

## Memory Index Entry Template

When story-architect creates or updates a project, it writes a memory file and index entry for cross-session navigation.

### Memory File Template

```markdown
---
name: story-architect: [Project Title]
description: Active story-architect project -- [genre_tag], last gate: [gate_name]
type: project
---

project_id: [slug]
title: [working title]
genre_tag: narrative-fiction | memoir | creative-nonfiction | long-form-essay
base_path: docs/writing/[slug]
handoff_path: docs/writing/[slug]/handoff/
last_gate_cleared: [gate name or "none"]
last_session: [ISO date]
session_mode: exploratory | rigorous | workshop
status: in_planning | handoff_written | drafting
```

### MEMORY.md Index Entry Format

```
- [story-architect: Project Title](project_[slug].md) -- [genre_tag], last gate: [gate]
```

### Memory Lifecycle

- **Created** at intake (project initialization).
- **Updated** at each gate clearance and at handoff generation.
- **Contains NO story content** -- only navigation pointers and status metadata.

---

## Escalation Payload Schemas

Two JSON payloads for skill-to-skill communication between story-architect and writing-guide.

### Direction 1: writing-guide escalates to story-architect

Triggered when writing-guide detects that the writer's needs have moved beyond craft guidance into structural planning territory.

```json
{
  "skill": "story-architect",
  "trigger": "structural-planning-needed",
  "context": {
    "detected_gap": "core_conflict_absent",
    "writer_mode": "exploratory",
    "active_project": "slug-if-known"
  }
}
```

| Field | Type | Notes |
|---|---|---|
| `skill` | string | Target skill identifier. Always `"story-architect"`. |
| `trigger` | string | Escalation reason. `"structural-planning-needed"`. |
| `context.detected_gap` | string | The specific structural gap that triggered escalation. |
| `context.writer_mode` | string | The writer's current mode at time of escalation. |
| `context.active_project` | string | Project slug if one exists, otherwise a sentinel value. |

### Direction 2: story-architect routes to writing-guide

Triggered when story-architect encounters a craft question that falls within writing-guide's domain rather than structural planning.

```json
{
  "skill": "writing-guide",
  "trigger": "craft-guidance-request",
  "context": {
    "planning_stage": "structure",
    "topic": "midpoint-shift",
    "kb_hint": "GUIDE.STRUCTURE.MIDPOINT_SHIFT.md",
    "intensity": "standard",
    "return_to": "story-architect"
  }
}
```

| Field | Type | Notes |
|---|---|---|
| `skill` | string | Target skill identifier. Always `"writing-guide"`. |
| `trigger` | string | Routing reason. `"craft-guidance-request"`. |
| `context.planning_stage` | string | Current gate or planning phase. |
| `context.topic` | string | The specific craft topic being routed. |
| `context.kb_hint` | string | Suggested KB file for writing-guide to consult. |
| `context.intensity` | string | Guidance depth: `"standard"` or other levels as defined. |
| `context.return_to` | string | Skill to return control to after guidance. Always `"story-architect"`. |

### Direction 3: prose-editor escalates to writing-guide

Triggered when prose-editor detects a problem that exceeds its editorial scope — voice drift requiring craft guidance, structural conflicts with locked planning artifacts, continuity contradictions, or premise-level diagnostic issues.

```json
{
  "skill": "writing-guide",
  "trigger": "voice-drift | structural-conflict | continuity-contradiction | diagnostic",
  "context": {
    "chapter": "[nn]-[slug]",
    "conflict_type": "[specific conflict category]",
    "description": "[factual description, no judgment]",
    "severity": "hold | flag | advisory",
    "review_artifact": "[path to review file]",
    "locked_gate_reference": "[gate name if relevant]"
  }
}
```

| Field | Type | Notes |
|---|---|---|
| `skill` | string | Target skill identifier. Always `"writing-guide"`. |
| `trigger` | string | One of: `"voice-drift"`, `"structural-conflict"`, `"continuity-contradiction"`, `"diagnostic"`. |
| `context.chapter` | string | Chapter identifier in `[nn]-[slug]` format. |
| `context.conflict_type` | string | Specific conflict category (e.g., "character-arc-violation", "timeline-contradiction"). |
| `context.description` | string | Factual description of the conflict. No judgment or severity language. |
| `context.severity` | string | One of: `"hold"`, `"flag"`, `"advisory"`. |
| `context.review_artifact` | string | Path to the review file containing the full finding. |
| `context.locked_gate_reference` | string | Gate name if the conflict involves a locked planning decision. Empty string if not applicable. |
