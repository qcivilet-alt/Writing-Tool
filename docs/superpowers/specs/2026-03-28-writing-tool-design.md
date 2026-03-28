# Writing Tool — story-architect Design Spec
updated: 2026-03-28
scope: story-architect module (first specialist skill)
status: complete — all 7 sections approved

---

## Section 1 — Executive Summary

The Writing Tool is a three-module, author-led writing support ecosystem built on Claude skills. This document specs the first specialist module: **story-architect**.

**System overview:**
- **writing-guide** (existing, foundational): craft mentor, best-practices guide, process guardian. Primary entry point and escalation hub.
- **story-architect** (this spec): structured story-creation workflow for writers building a story from scratch. A planning coach and story architect — not a co-author.
- **prose-editor** (future spec): editorial review and continuity layer for writers actively drafting or revising.

**Confirmed design decisions:**
- **Architecture:** Layered ecosystem — writing-guide as hub, specialists as extensions via skill-to-skill API
- **Genre scope:** Explicitly genre-agnostic — a structural abstraction layer handles both narrative fiction and long-form non-fiction (memoir, essay, creative nonfiction), with the same framework adapting to genre rather than requiring genre-specific routing
- **Friction model:** Writer-set intensity at session start (Exploratory / Rigorous / Workshop), not auto-escalated
- **Persistence:** File-based artifacts as ground truth (`docs/writing/[project]/`), memory index as session pointer
- **Anti-ghostwriting:** Non-bypassable. All module outputs are structural artifacts, annotations, questions, or single-sentence suggestions tied to writer-provided input. No module generates prose from a description.

**Core principle:** The writer is the author. The tool is the architect's table — it holds plans, marks gaps, and asks hard questions, but it never picks up the pen.

---

## Section 2 — Use Case Breakdown: story-architect

### Primary Purpose

Help writers move from raw, vague ideas to a formalized story framework — a structured set of persistent artifacts that can guide the actual drafting process. The module makes writers think more rigorously and deliberately before committing to sustained prose work. It does this by detecting structural gaps via a 3-pass scan, firing targeted friction questions in priority order, holding planning gates until required fields are resolved, and saving all writer-provided decisions as versioned artifacts.

---

### Ideal User Scenarios

- "I have an idea but don't know where to start"
- "I have a character but no plot to put them in"
- "I started writing but lost direction — I want to go back and plan it properly"
- "I want to check if my premise actually works before I spend months on it"
- "I'm writing a memoir and I can't find the through-line"
- "I have twenty pages of notes and need to organize them into something"

---

### Expected Inputs

- Raw story idea (can be vague or fragmentary)
- Premise fragments, character concepts, tone/genre intentions
- Existing notes or scattered drafts (handled via structured intake, not dump-and-proceed)
- **Genre tag** selected at session start: `narrative-fiction` / `memoir` / `creative-nonfiction` / `long-form-essay`
- **Session mode** selected at session start: Exploratory / Rigorous / Workshop

---

### In-Session Interventions

*Conversational only — exist in the chat context, never written to disk.*

| Intervention | Fires When |
|---|---|
| **Intake Prompt** | Session start / project unresolved |
| **Friction Question** | Gap scan detects a missing/underdeveloped required field — one per turn, highest priority |
| **Gap Flag** | Missing or blank element found in artifact scan |
| **Coherence Alert** | New writer input conflicts with something already recorded in an artifact |
| **Gate Prompt** | Phase boundary reached — structured yes/no confirmation |
| **Gate Confirmation** | Writer confirms gate — names what was confirmed and what artifact action follows |
| **Artifact Write Notice** | Immediately before every file write — names the file and what changed |
| **Artifact Write Receipt** | Immediately after every file write — confirms path |
| **Session Summary** | Writer signals session end |
| **Handoff Prompt** | Writer signals stop/switch — captures intent, next scene, open questions before writing `handoff-context.json` |

**Invariants:**
- No file is written without a preceding Notice and following Receipt
- No file contains content not sourced from writer input
- If the skill detects it is about to write generated content, it halts and fires a friction question instead

---

### Persistent Artifacts

*Written to `docs/writing/[project]/` — survive the session, writer-owned.*

| Artifact | Location | Contents | Written When | Write Trigger |
|---|---|---|---|---|
| `story-manifest.md` | `docs/writing/[project]/` | Logline, premise, genre tag, core conflict, thematic argument, tone markers, gate clearance log | Created at intake gate; updated at each gate confirmation | Auto on gate; confirm on update |
| `structure-map.md` | `docs/writing/[project]/` | Act/part breakdown, major turning points (writer's words), pacing notes, open structure questions | Created when structure gate confirmed | Auto on gate; confirm on update |
| `characters/[name].md` | `docs/writing/[project]/characters/` | Character frame (genre-appropriate — see below), key relationships, unresolved questions | Created per character when character gate confirmed | Auto on gate; confirm on update |
| `chapters/[nn]-[slug].md` | `docs/writing/[project]/chapters/` | Chapter number, POV, writer-provided scene beats, entry/exit state, open questions | Created when chapter plan confirmed | Auto on gate; confirm on update |
| `handoff-context.json` | `docs/writing/[project]/handoff/` | Structured JSON: gates cleared, open questions, artifact index, session mode — for prose-editor intake | Written at session end / handoff prompt | Auto on handoff prompt completion |
| `[project]-kb-patch.md` | `docs/writing/[project]/` | Writer-confirmed world rules, recurring terms, project-specific additions | When writer explicitly approves a KB addition | Explicit writer approval required |

---

### Friction Trigger Model

The skill runs a **3-pass gap scan** on the accumulated story brief each turn, before deciding whether to fire a friction question.

**Pass 1 — Field Emptiness**
Each required field has a canonical slot. The scan checks whether the slot is empty, null, or contains only a placeholder phrase. Placeholder detection uses a blocklist: `TBD`, `not sure yet`, `something about`, `maybe`, single-word entries for fields requiring a clause.

**Pass 2 — Underdevelopment**
A populated field is not necessarily developed. Three sub-checks:
- *Clause threshold:* field must contain at least one subject + verb construction ("revenge" fails; "the protagonist wants revenge against her father" passes)
- *Specificity threshold:* proper nouns, concrete objects, or named relationships must appear at least once per major character field
- *Internal completeness:* certain fields require paired sub-fields (e.g., `structure` requires both opening condition and ending condition)

**Pass 3 — Logical Contradiction**
Checks for conflicts between populated fields:
- Want/Obstacle mismatch: stated want has no plausible obstacle
- Stakes/Ending mismatch: high stated stakes but resolution has no cost
- Theme/Story mismatch: stated theme not traceable to any character arc or plot event
- Tone/Structure mismatch: light tone with no humor beats or release valves in the described structure

Contradictions outrank emptiness in priority. A populated-but-contradictory field is more dangerous than a blank one.

**Priority + Selection Rule**

Step 1 — Assign severity: contradiction = 3, empty = 2, underdeveloped = 1.
Step 2 — Apply category priority within each severity tier: Premise → Character → Structure → Theme.
Step 3 — Apply recency suppression: if the writer answered a question about this field in the last two turns, move it to the bottom of the queue regardless of severity.
Step 4 — Fire exactly one question. Top of ranked list. Log the field and turn number.

**Friction Trigger Tables**

*Premise triggers:*

| Condition | Question |
|---|---|
| `core_conflict` empty or placeholder | "What force or situation is actively working against what your [protagonist] wants most?" |
| `core_conflict` has no opposing agent | "Who or what specifically stands between your [protagonist] and what they're after — and why can't they simply step aside?" |
| `stakes` empty | "If your [protagonist] fails completely, what is permanently lost or broken?" |
| `stakes` present but reversible with no cost | "What in your story makes that loss irreversible?" |
| `inciting_event` empty or lacks specific trigger | "What specific event forces your [protagonist] into this story rather than letting them stay in their ordinary life?" |

*Character triggers:*

| Condition | Question |
|---|---|
| `want` empty or abstract noun only | "What does your [protagonist] actively want at the start of this story — stated as a specific goal?" |
| `need` empty | "What does your [protagonist] actually need that they don't yet know they need?" |
| `want` and `need` identical | "If want and need are the same thing, what creates the internal tension?" |
| `wound/backstory_driver` empty | "What happened to your [protagonist] before this story begins that makes them exactly wrong — or exactly right — for this situation?" |
| `change_vector` empty | "Where does your [protagonist] end up internally that they couldn't have reached at the beginning?" |

*Structure triggers:*

| Condition | Question |
|---|---|
| `opening_condition` empty | "What is the specific state of your [protagonist]'s world at the moment this story begins?" |
| `ending_type` empty | "Do you know yet how this story ends — not plot mechanics, but the emotional or thematic resolution?" |
| `midpoint_shift` empty | "What happens in the middle that reframes what the story is actually about — not just escalates it?" |
| `ending_type` contradicts `stakes` | "Your ending doesn't seem to cost your [protagonist] anything — what does the resolution actually demand from them?" |

*Theme triggers:*

| Condition | Question |
|---|---|
| `central_question` empty | "What question does this story hold open — not answer, but genuinely sit with — by the last page?" |
| `central_question` is a statement | "Can you reframe this as a question the story refuses to answer cleanly?" |
| `traceable_event` empty | "Where in the actual events of your story does this theme get tested most severely?" |
| `central_question` conflicts with `change_vector` | "Your theme asks one question but your [protagonist]'s arc answers a different one — which is this story actually about?" |

**Dormant Rule — 5 conditions when no question fires:**
1. Intake still in progress (minimum viable brief not yet established)
2. Writer just answered a friction question (question answered in prior turn — still in "unanswered" queue)
3. Writer asked the skill a direct question (answer it; don't redirect with friction)
4. Writer declared a generative sprint ("I'm going to brainstorm for a bit") — skill enters listener mode until sprint ends
5. Brief is sufficiently developed — all required fields populated above clause and specificity thresholds with no contradictions

**Genre Vocabulary Adaptation**

Triggers are identical across genres. Surface language adapts via substitution:

| Generic | narrative-fiction | memoir | creative-nonfiction | long-form-essay |
|---|---|---|---|---|
| protagonist | protagonist | narrator / I | subject / central figure | author's position |
| want | goal | driving question | investigative lens | central argument |
| wound | backstory wound | formative experience | entry point | animating tension |
| arc | character arc | arc of understanding | arc of inquiry | movement of thought |
| inciting event | inciting incident | rupture or trigger | precipitating event | the problem that won't resolve |
| stakes | story stakes | personal stakes | stakes of the record | stakes of the claim |
| theme | theme | meaning made | frame of significance | thesis territory |

---

### Genre Abstraction Layer

Same gates, friction engine, and artifact system across all genres. Character frame vocabulary adapts to genre tag.

**Narrative Fiction — Character Frame (want / need / misbelief / arc)**
The established KB framework. Backed by `GUIDE.CHARACTER.WANT_NEED_MISBELIEF.md`.

**Memoir — Character Frame**

| Concept | Fiction analogue | Definition | KB backing |
|---|---|---|---|
| **The Wound** | Want | The specific, named injury organizing the narrator's pursuit — not "difficult childhood" but a precise rupture | GUIDE.CHARACTER.WANT_NEED_MISBELIEF.md (analogue) |
| **The Reckoning** | Need | What the narrator must confront to reach understanding — often different from what they think they want, only visible in retrospect | GUIDE.CHARACTER.AGENCY_AND_DECISION_PRESSURE.md |
| **The Lie the Self Told** | Misbelief | The survival story the narrator held that organized their behavior and prevented clear sight — must have once been genuinely seductive | GUIDE.CHARACTER.WANT_NEED_MISBELIEF.md (analogue) |
| **The Earned Distance** | Arc | The epistemological shift in the narrator's relationship to their own story — not necessarily healing, but changed altitude of seeing | `GUIDE.CRAFT.MEMOIRIST_DISTANCE.md` (new KB file needed) |

**Creative Nonfiction — Character Frame**

| Concept | Fiction analogue | Definition |
|---|---|---|
| **The Obsession** | Want | The specific, driving compulsion that sent the writer toward this subject — not "I was curious" but "I could not stop thinking about this" |
| **The Hidden Claim** | Need | The argument the material forces, often different from the stated thesis — what the piece must say to be true |
| **The Reporter's Complicity** | Misbelief | The assumption or bias the writer brought that the material must expose — what they wanted to be true, and what reality disrupted |
| **The Form as Argument** | Arc | How structure and form enact the piece's meaning — the reader's journey, not the writer's change |

**Long-Form Essay — Character Frame**

| Concept | Fiction analogue | Definition |
|---|---|---|
| **The Animating Question** | Want | The specific, genuinely open question the essay attempts — not a topic, not a thesis, but a real inquiry the writer does not yet have the answer to |
| **The Persona** | Need | The constructed version of the self the essay performs — the facet of intelligence this inquiry requires the essayist to become |
| **The Governing Contradiction** | Misbelief | The paradox or irresolvable tension the essay holds in suspension — unlike fiction's misbelief, this may remain unresolved at the end (genre-defining, not a gap) |
| **The Movement of Mind** | Arc | The traceable cognitive journey of the essayist's thinking — where they start, how they pursue, where they arrive or deliberately fail to arrive |

**Sharpest structural difference across genres:** Fiction misbelief is overcome. Memoir's lie is exposed. CNF's complicity is interrogated. Essay's governing contradiction **may be held permanently** — this is genre-normative, not a design gap. Gate questions for the misbelief-equivalent must account for this.

---

### Session Modes

**Mode selected by writer at session start. Can be changed at any resume point.**

**EXPLORATORY** *(default)*
- **Opening:** One orienting question. Names the mode briefly. Does not enumerate agenda. Writer sets direction.
- **Question style:** Curious, lateral, non-evaluative. Follows writer's thread. "What's the part you keep coming back to?" not "Have you defined your protagonist's wound?"
- **Gap handling:** Noted lightly, offered as optional exploration. Writer can decline. Gaps logged but never block progress.
- **Stuck response:** Treats "I don't know" as useful. Offers to explore, park, or use constraint as prompt. Never pushes.
- **Positive feedback:** Reflective only. Reflects back what it heard. No grading.
- **Escalation:** Suggests Rigorous only if structural gaps are likely to become costly. Once, no pressure, not repeated.
- **Tone:** Spacious, attentive, unhurried, lateral, non-directive.
- **Choose when:** Still finding the story — fragments, impulses, no structure yet. Also good for re-entry after time away.

**RIGOROUS**
- **Opening:** Status check → session stage → active gates. Writer confirms focus before proceeding. Working meeting, not an examination.
- **Question style:** Direct, sequential, builds toward assessment. "What does your protagonist lose if they succeed?" not "What's at stake?" Follow-ups push for precision. One question per turn, legible sequence.
- **Gap handling:** Named explicitly, given weight, tracked. Significant gaps at a gate block advancement with a specific reason stated.
- **Stuck response:** Distinguishes "haven't thought about it" from "it isn't working." Diagnoses type, offers targeted prompts. At gate-critical questions, can park the element but flags it as unresolved.
- **Positive feedback:** Brief, specific, proportionate. "That clarifies the antagonist's motivation — it was the missing piece in Act Two's logic." Not withheld, not inflated.
- **Escalation:** Suggests Exploratory if premise is still fundamentally unstable. Suggests Workshop if answers are consistently strong and work is at execution level.
- **Tone:** Precise, structured, engaged, purposeful, exacting.
- **Choose when:** Working premise, protagonist, rough story shape — need to stress-test structure before drafting at scale.

**WORKSHOP**
- **Opening:** Formal session frame: states mode, scope, evaluation criteria, expected output. Writer confirms readiness. Functions as an explicit contract.
- **Question style:** Socratic, adversarial in the classical sense. Follows writer's answer to its logical conclusion and tests whether it holds. Returns to the same question from a different angle if the first answer didn't resolve the tension.
- **Gap handling:** Gaps treated as claims requiring a response. States downstream consequences. Response required before advancing. Unacknowledged gaps are session obligations, not soft suggestions.
- **Stuck response:** Asks writer to characterize the stuckness (knowledge gap / values conflict / structural impossibility / other). Asks writer to articulate what a resolved version would look like. May recommend ending the session or switching mode if stuckness is unproductive.
- **Positive feedback:** High bar, structural, earned. "That works because it resolves both the motivation gap and the thematic throughline simultaneously." Writer knows Workshop acknowledgment means something.
- **Escalation:** Suggests Rigorous if Workshop friction is producing avoidance rather than productive tension — named once with a specific observation.
- **Tone:** Rigorous, deliberate, adversarial (productively), demanding, collegial.
- **Choose when:** Story feels structurally sound and writer wants to find out if they're right. Close to draft completion and need a reader who pushes back on logic, not just identifies gaps.

**Key distinction between Rigorous and Workshop:** Rigorous asks *"is this resolved?"* Workshop asks *"does this hold under contradiction?"* Rigorous moves through a gap checklist. Workshop interrogates the internal logic of answers the writer already has.

---

### Gap-to-Gate-Hold Rule

**The 3-condition escalation rule:**

A gap flag becomes a gate hold when all three conditions are true:
1. The field is in the **required set** for the current gate × genre tag
2. The session mode is **Rigorous or Workshop**
3. The gap severity is **Critical or Structural**

In Exploratory mode: no gap ever becomes a gate hold. All gaps annotate as `[UNRESOLVED: reason]` in the relevant artifact field.

In Rigorous mode: required fields cannot be skipped. Module holds and re-asks.

In Workshop mode: gate hold raised, but a **deferral path** is available — writer provides a 6-field justification (`field`, `gate`, `reason_type`, `reason_statement`, `resolution_trigger`, `risk_acknowledgment`). Deferrals expire at Chapter Direction. A `force_defer` override exists but triggers a session-level warning if invoked more than twice.

**Gap Severity Model:**

| Severity | Definition | Examples | Behavior in Rigorous | Behavior in Workshop |
|---|---|---|---|---|
| **Critical** | Absence makes gate's downstream outputs logically undefined | `core_tension`, `act_structure`, `point_of_view_confirmed` | Gate hold, no override | Gate hold; deferral requires `reason_type: structural-experiment` + coherence review |
| **Structural** | Creates dependency risk — downstream work possible but expensive to fix later | `character_arc_type`, `time_structure`, `argument_arc` | Gate hold | Gate hold with deferral path |
| **Advisory** | Quality signal, not a structural dependency | `cast_necessity_check`, `tonal_consistency_check` | Never a gate hold | Never a gate hold — listed as craft notes |

Severity is assigned from a static gate-map configuration (defined per gate × genre), not computed dynamically. Writers know in advance which gaps will be critical.

**Gate Map — Required Fields (summary):**

| Gate | narrative-fiction (required) | memoir (required) | long-form-essay (required) |
|---|---|---|---|
| **Premise Lock** | `core_tension`, `protagonist_stake`, `thematic_question`, `inciting_premise` | `core_tension`, `protagonist_stake`, `thematic_question`, `narrative_voice` | `core_tension`, `thematic_question`, `narrative_voice`, `argument_claim` |
| **Character Commitment** | `protagonist_desire`, `protagonist_wound`, `antagonist_function`, `character_arc_type` | `protagonist_desire`, `protagonist_wound`, `narrator_stance`, `key_relationships_mapped` | `narrator_stance`, `speaker_ethos` |
| **Structure Approval** | `act_structure`, `inciting_incident_placed`, `midpoint_reversal`, `climax_sketch` | `act_structure`, `climax_sketch`, `time_structure` | `act_structure`, `chapter_count_range`, `section_logic`, `argument_arc`, `tonal_consistency_check` |
| **Chapter Direction** | `scene_objective`, `point_of_view_confirmed`, `entry_exit_beats`, `tension_raised`, `character_state_delta` | `scene_objective`, `point_of_view_confirmed`, `character_state_delta`, `sensory_anchor` | `chapter_argument`, `point_of_view_confirmed`, `factual_research_flag`, `transitional_logic` |

---

### Handoff Context Block

Written to `docs/writing/[project-id]/handoff/handoff-context-v[N].json` when the writer is ready to draft. Read by prose-editor at session start.

**Schema summary:**

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
    "author_note": "optional, max 300 chars — tone-setter from writer to prose-editor"
  },
  "gates": {
    "premise":        { "status": "locked|provisional|skipped|not_started", "locked_at": "", "note": "" },
    "structure":      { "...": "..." },
    "characters":     { "...": "..." },
    "world_or_context": { "...": "..." },
    "scene_sequence": { "...": "..." },
    "theme_statement": { "...": "..." },
    "voice_profile":  { "...": "..." },
    "extended":       {}
  },
  "artifacts": {
    "premise_doc":        { "path": "", "format": "", "description": "", "last_modified": "" },
    "outline":            { "...": "..." },
    "character_profiles": [],
    "voice_doc":          { "...": "..." },
    "drafts_dir":         "drafts/",
    "extended":           {}
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

**Versioning:** MAJOR.MINOR. MAJOR bump on any breaking change (renamed/removed required field). MINOR bump on backward-compatible additions. `extended` buckets are the safe expansion surface. Prose-editor refuses to load a mismatched MAJOR and surfaces a human-readable error.

**Four things prose-editor must treat as load-bearing:**
1. `project.genre_tag` — the editorial lens selector, first field acted on
2. `gates` — locked gates are hard constraints, not suggestions
3. `artifacts` — file index; prose-editor loads from these paths, never from the handoff block itself
4. `open_questions` + `gate_surfacing` — together govern when prose-editor interrupts the writer

---

### Multi-Session Resume Behavior

**Session start decision tree (5 steps):**

1. Read memory index → no pointer = first session / import flow; pointer found → Step 2
2. Read `handoff-context.json` → missing/invalid = degraded resume (see below)
3. Read `story-manifest.md` → missing = degraded resume
4. Evaluate `last_session_status`: `mid_gate` / `between_gates` / `project_complete` / `intake_incomplete`
5. Present resume summary → writer confirms / changes mode / requests gate revision / switches project

**Resume summary template** *(populated from artifacts only — never from memory or inference)*:

```
STORY-ARCHITECT — Resuming: [Project Title]
Session [N] · [Mode] · Last active: [relative date]

YOUR STORY SO FAR
Genre:        [value or "not set"]
Premise:      [one-sentence or "not established"]
Protagonist:  [name + phrase or "not developed"]
Core tension: [value or "not established"]

DEVELOPMENT STATUS
✓ [Gate]  cleared [date]
→ [Gate]  IN PROGRESS  ← last position
○ [Gate]  pending

LAST SESSION
Worked on:    [1–2 sentences]
Stopped at:   [gate + step]
Open threads: · [thread 1]  · [thread 2]  (max 4)

ARTIFACTS ON FILE
story-manifest.md     present / MISSING
characters/           [N] files / empty
structure-map.md      present / MISSING
chapters/             [N] files / empty
```

**Stale artifact tiers:**
- **Tier 1** (story-manifest or handoff-context missing): halt, offer: search / rebuild / new project
- **Tier 2** (non-critical file missing): proceed, flag as MISSING, add to open threads
- **Tier 3** (timestamp drift): flag as potentially out of date, reconcile when relevant
- **Tier 4** (parse error): attempt raw read, treat as Tier 1 or 2 by file type

**Multi-project:** If memory index contains more than one project pointer, skill always asks for selection before loading. Never assumes. If one project has `last_session_status == "mid_gate"`, surfaces that fact in the list.

**Mid-gate resume:** Shows the exact question that was interrupted and any partial response verbatim. Offers: continue / start gate question over / skip (non-required only) / abandon gate.

**Gate revision:** Always available. Skill surfaces downstream consequences before proceeding, presents diff after completion, does not auto-cascade — identifies affected artifacts and asks writer to decide. Revision history tracked (array, not overwrite).

**Mode change on resume:** Upgrade shows what's added + confirms. Downgrade shows what's removed + warns. Never re-runs completed gates — applies prospectively only. Recorded in mode_history array.

---

### What story-architect Should Do

- Run a 3-pass gap scan each turn and fire one friction question targeting the highest-priority gap
- Hold at gates in Rigorous/Workshop until required fields are resolved per the gate map
- Save artifacts after each gate clearance with a preceding Write Notice and following Receipt
- Log all unresolved gaps persistently as `[UNRESOLVED: reason]` until addressed or deferred
- Adapt structural concepts and character frame to the writer's genre tag
- Resume correctly from any prior session state using the decision tree
- Surface downstream consequences before any gate revision or mode change
- Apply friction and tone calibrated to the writer's chosen session mode

### What story-architect Must Not Do

- Generate plotlines, backstories, or chapter content to fill gaps
- Write story summaries from vague descriptions
- Advance past a gate in Rigorous/Workshop without writer-provided answers
- Validate a weak premise to be encouraging
- Fire more than one friction question per response
- Write any file without a preceding Artifact Write Notice
- Make decisions the writer hasn't made — only surface, organize, and hold
- Invoke Workshop-level friction in Exploratory mode regardless of gap severity

---

## Section 3 — Key Features: story-architect

*What makes this module work in practice. Focuses on new detail — Section 2 covers gate rules, mode behavior, friction triggers, and resume flow.*

---

### Feature 1 — The Gate System

Gates are the structural backbone of story-architect. They enforce the principle that planning decisions have dependencies — you cannot commit to a structure without a premise, and you cannot plan chapters without a structure.

**Why gates, not an open canvas:** An open canvas lets writers plan in any order, which sounds like freedom but produces incoherence. A writer who plans chapters before committing to a premise will plan chapters that don't serve the premise. Gates introduce the right dependencies in the right order. They don't restrict creativity — they sequence commitment so that earlier decisions constrain later ones productively.

**The gate sequence:**

```
INTAKE
  └─ story-manifest.md stub created

PREMISE LOCK
  └─ Core tension, stakes, thematic question confirmed
  └─ story-manifest.md written in full

CHARACTER COMMITMENT (repeats per principal character)
  └─ Full character frame confirmed for each character
  └─ characters/[name].md written per character

STRUCTURE APPROVAL
  └─ Macro architecture confirmed (acts, arc, key turns)
  └─ structure-map.md written

CHAPTER DIRECTION (repeats per chapter)
  └─ Per-chapter outline shell confirmed
  └─ chapters/[nn]-[slug].md written per chapter

HANDOFF
  └─ handoff-context.json written
  └─ prose-editor intake can begin
```

**What happens within a gate — the interaction flow:**

Each gate is a conversation, not a form. The skill does not present a list of required fields and ask the writer to fill them in. It asks one friction question at a time, evaluates the answer, updates the relevant artifact field, and either asks the next question or signals gate readiness.

```
GATE OPENS
  └─ Skill states which gate is active and why it matters
  └─ Friction engine runs gap scan on current brief
  └─ First friction question fires on highest-priority gap

WRITER ANSWERS
  └─ Skill evaluates answer against minimum-viable resolution standard
  └─ [Pass] → Field updated in artifact, next gap scanned, next question fires
  └─ [Fail] → Answer validation response (see Feature 2), same question re-framed

ALL REQUIRED FIELDS RESOLVED
  └─ Skill signals gate readiness: "All required fields for [gate] are resolved."
  └─ Gate Prompt fires: "Ready to lock [gate]?"

WRITER CONFIRMS
  └─ Gate Confirmation fires
  └─ Artifact Write Notice → artifact written → Artifact Write Receipt
  └─ Gate clearance logged in story-manifest.md
  └─ Next gate opens
```

**Genre tag at intake:** Genre tag is set at session intake and governs all gate required-field sets and character frame vocabulary. Before Premise Lock is cleared, the writer can restart intake to change the tag. After Premise Lock, changing the tag requires the gate revision flow because required fields for Character Commitment and Structure Approval were set under the prior tag. Hybrid genre handling is out of scope for v1 — the writer selects the closest primary tag and notes the hybrid nature in `story-manifest.md` as a freeform field. One tag governs all gate requirements.

---

### Feature 2 — Friction Engine

The friction engine is what separates story-architect from a planning template. A template asks the writer to fill in fields. The friction engine interrogates the writer's answers — it evaluates, pushes back, and holds until the field is genuinely resolved.

**Why one question per turn:** Asking multiple questions simultaneously transfers the prioritization burden to the writer. The skill's job is to know which gap matters most right now and ask only that one. A list of questions also lets the writer answer the easiest ones and skip the hard ones. One question enforced makes avoidance visible.

**Why questions are probes, not prompts:** A prompt ("describe your protagonist's wound") invites a correct-sounding answer that may not reflect actual story thinking. A probe ("what happened to your protagonist before this story begins that makes them exactly wrong — or exactly right — for this situation?") requires the writer to locate the answer in their own understanding of the character. Probes can't be satisfied by template-filling.

**Answer validation and rejection:**

When a writer's answer fails minimum-viable resolution (still a placeholder, still a fragment, no subject+verb construction, no specificity), the skill does not silently accept it and move on. The response follows this pattern:

```
[What was heard] — restates the writer's answer in one phrase
[What's missing] — names the specific gap the answer didn't close
[Re-framed probe] — asks the question from a different angle
```

Example:

```
You said: "She wants revenge."
That's a feeling, not a goal — it doesn't tell us what she's trying to make happen
in the world. What specific action is she trying to take, and what would success
look like from the outside?
```

The skill does not generate an example of a resolved answer. That would be ghostwriting. It can describe the structure of a resolved answer ("a specific action, a named target, a concrete consequence") without providing one.

**Answer validation re-ask limit:** After three failed attempts at the same field, the skill names the pattern directly: "We've circled this question three times. It may not be answerable yet — do you want to park it and come back, or explore what's blocking you?" This prevents the session from stalling in a loop.

**Generative sprint mode:**

A writer can declare a sprint — a period of open thinking, brainstorming, or stream-of-consciousness that is not subject to friction. The friction engine goes dormant for the sprint's duration.

*Entry trigger:* Writer says any variation of "I want to think out loud," "let me brainstorm for a bit," "I'm going to explore this for a minute," or "sprint."

*Behavior during sprint:* Skill enters listener mode. It reflects back key phrases, tracks new story material for later gap scanning, and does not challenge or question. It does not log gap flags during a sprint — the material is exploratory, not commitments.

*Exit trigger:* Writer says "done," "okay, what did you notice?", "let's check what we have," or any explicit return signal. Skill summarizes what emerged from the sprint, identifies any new fields that were populated or implied, and resumes the friction engine from the current gap scan.

*Why sprint mode exists:* Some story problems need to be talked through before they can be interrogated. Forcing friction on exploratory thinking interrupts the creative process before it produces anything worth interrogating. Sprint mode creates a protected generative space within a structured session.

---

### Feature 3 — Gap Annotation System

When a required or advisory field is absent or underdeveloped, the skill annotates it inline in the relevant artifact:

```markdown
protagonist_want: [UNRESOLVED: want stated as feeling ("revenge") not as goal — re-ask]
```

**Resolution semantics — exactly when an annotation is removed:**

An annotation is removed when:
1. The writer provides an answer that passes minimum-viable resolution (complete sentence, no placeholder tokens, meets clause and specificity thresholds), **and**
2. The skill has processed the answer (gap scan on the updated field returns no flag), **and**
3. One of: (a) the gate is still open and the field was re-evaluated in the current session, or (b) the writer explicitly confirms the field during a gate clearance.

An annotation is **not** removed:
- When the writer manually edits it out of the artifact file (the skill re-adds it on next scan unless the field also passes resolution)
- When the writer answers with another placeholder
- Automatically by the passage of time

**Why annotations persist until resolution:** Silent gap removal would allow the writer to paper over structural holes. Annotations are the skill's memory of where the plan is not yet sound. They are not accusations — they are markers that say "this needs work before this decision can be trusted."

**Annotation ownership:** Annotations are always written by the skill, not by the writer. If the writer edits an artifact file directly and adds content to an annotated field, the skill re-evaluates the field on next scan and either removes the annotation (if the content passes) or updates the annotation to reflect the new state of the field.

---

### Feature 4 — Artifact Generation Pipeline

Artifacts are built incrementally — never generated in a bulk pass from a thin description. Each artifact starts as a stub at intake and gains fields only as the writer provides content and the relevant gate is cleared.

**Why incremental over bulk:** A bulk-generated artifact based on a vague description would require the skill to invent content — ghostwriting by another name. Incremental generation means every field in every artifact traces to a specific writer-provided answer in a specific session turn. The artifact is the writer's thinking, organized by the skill — not the skill's thinking presented as the writer's.

**Stub → full progression:**

```
INTAKE
  └─ story-manifest.md stub:
     project, genre_tag, session_mode, created_date
     all story fields: null with [UNRESOLVED] annotations

PREMISE LOCK CLEARED
  └─ story-manifest.md gains:
     logline, core_tension, protagonist_stake, thematic_question, tone_markers
     gate_log: premise_lock: [date]
     [UNRESOLVED] annotations on resolved fields removed

CHARACTER COMMITMENT CLEARED (per character)
  └─ characters/[name].md created:
     genre-appropriate character frame (all fields, writer-provided)
     unresolved fields remain annotated

STRUCTURE APPROVAL CLEARED
  └─ structure-map.md created:
     act breakdown, key turns, pacing notes (writer-provided)
  └─ story-manifest.md updated:
     gate_log: structure_approval: [date]

CHAPTER DIRECTION CLEARED (per chapter)
  └─ chapters/[nn]-[slug].md created:
     scene beats, POV, entry/exit state, open questions (writer-provided)

HANDOFF
  └─ handoff-context.json written:
     full artifact index + gate status + open_questions list
```

**What the writer sees at each write:** Every artifact write is preceded by an Artifact Write Notice naming the file and stating what changed. Every write is followed by an Artifact Write Receipt confirming the path. This makes the artifact pipeline visible — the writer always knows what was saved and where.

---

### Feature 5 — Integration with writing-guide

Story-architect and writing-guide are distinct skills with a defined division of labor: story-architect plans, writing-guide teaches craft. Neither crosses into the other's domain.

**Why the division matters:** A skill that tries to both plan and teach will do both poorly. The planning session will stall on craft tangents; the craft guidance will be distorted by the planning context. Clean separation lets each skill do what it's built to do and hand off cleanly when the writer needs the other.

**Direction 1 — writing-guide escalates to story-architect:**

When a writer in writing-guide surfaces a structural planning need (weak premise, no character arc, no causal plot logic), writing-guide routes outward via the skill-to-skill API:

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

Story-architect receives this context, skips generic intake, and opens directly at the Premise Lock gate with the relevant friction question loaded.

**Direction 2 — story-architect routes to writing-guide:**

When a writer in story-architect asks a craft question, story-architect does not answer it. The response is:

```
That's a craft question — routing to writing-guide.
[writing-guide response to: GUIDE.PREMISE.VIABILITY.md / intensity: standard]
Back to planning. We were working on [field]. Ready to continue?
```

The session does not end. The writer sees the craft answer inline, then story-architect resumes with a bridge.

**Escalation API:**

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

The `return_to` field signals writing-guide to end with a bridge back to the planning context.

---

### Feature 6 — Author-Control Safeguards

The following controls are always in the writer's hands:

| Control | What the writer can do |
|---|---|
| Session mode | Change at any session start or resume point — upgrade/downgrade at will |
| Genre tag | Change before Premise Lock by restarting intake; after Premise Lock via gate revision flow |
| Gate revision | Request revision of any cleared gate at any time — consequences surfaced, diff shown, no auto-cascade |
| Sprint mode | Declare a generative sprint — friction engine goes dormant until explicitly ended |
| Deferral (Workshop only) | Defer a required gap with a 6-field written justification |
| Artifact review | Request to see any artifact in full at any time during a session |
| Session end | Stop at any point — all progress saved automatically before the session ends |

The following are enforced by the skill and not overridable by the writer:

| Constraint | Why it exists |
|---|---|
| Anti-ghostwriting — no file written with generated content | The artifact is the writer's thinking organized by the skill, not the skill's thinking presented as the writer's. Allowing generated content would make artifacts unreliable as a record of the writer's actual decisions. |
| Artifact Write Notice precedes every file write | Writers must know what is being written to their filesystem on their behalf. Silent writes break trust and make artifacts hard to audit. |
| One friction question per turn | The skill's job is to prioritize — presenting multiple questions transfers that burden to the writer and allows avoidance of the hard questions. |
| Gate hold in Rigorous/Workshop on required fields | The gates exist to create load-bearing dependencies. Skipping a required field means building later decisions on an unstated assumption, which produces plans that collapse during drafting. |
| Memory never overrides artifact file content | Memory is a session index, not a content store. Trusting memory over files would create silent data drift — the skill would operate on outdated information without the writer knowing. |
| Annotation persists until field passes resolution | Silent gap removal allows structural holes to be papered over. Annotations are the skill's record of where the plan is not yet sound — they are removed only by genuine resolution. |

---

## Section 4 — Memory and Control Structure

---

### 4.1 — The Two-Layer Storage Model

Story-architect stores everything in two layers that serve different purposes and never conflict.

**Layer 1 — Artifact files (ground truth)**
All planning decisions live in files in `docs/writing/[project]/`. These persist when Claude is closed, when session context is lost, when memory is cleared, or when the writer switches sessions. They are owned by the writer — not by the skill.

**Layer 2 — Memory index (session pointer)**
The memory index holds a pointer to the active project so story-architect can locate its artifact files without asking the writer where they are. The memory index contains no story content — only paths and last-known gate state.

**The conflict rule:** If memory index state and artifact file content disagree, the artifact file wins. Memory is a navigation aid, not a source of truth. When a conflict is detected, the skill reads the file, updates the memory pointer to match, and surfaces the discrepancy: "My session pointer said [X], but the file says [Y] — using the file."

**Why this split:** Storing story content in memory would make it fragile and invisible. Memory can be summarized away, cleared, or corrupted; artifact files cannot. Memory's only job is to remove file path management friction from session start.

---

### 4.2 — The Artifact Schema

**Directory structure:**

```
docs/writing/
└── [project-slug]/
    ├── story-manifest.md          ← project identity, gate log, premise decisions
    ├── structure-map.md           ← macro architecture
    ├── cross-character-relationships.md  ← cross-character dynamics (single source of truth)
    ├── [project-slug]-kb-patch.md ← project-specific world rules (optional)
    ├── characters/
    │   └── [character-slug].md   ← one file per principal character
    ├── chapters/
    │   ├── [nn]-[slug].md        ← story-architect planning artifact
    │   └── [nn]-[slug]-review.md ← prose-editor output (separate, never overwrites planning)
    ├── handoff/
    │   ├── handoff-context-v1.json  ← point-in-time snapshots for prose-editor intake
    │   └── handoff-context-v2.json  ← versioned on each Chapter Direction revision
    └── checkpoints/
        └── [YYYY-MM-DD]-manifest.md  ← snapshots before major structural changes (v2)
```

**What each artifact holds (category descriptions — exact field schemas live in `references/artifact-schemas.md`):**

| Artifact | Category of information | Written by | Read by |
|---|---|---|---|
| `story-manifest.md` | Project identity, premise decisions, gate log, open threads | story-architect | writing-guide, prose-editor |
| `structure-map.md` | Macro architecture: act breakdown, key structural turns, pacing notes | story-architect | prose-editor |
| `cross-character-relationships.md` | Cross-character dynamics: relationship type, dynamic, status | story-architect | prose-editor |
| `characters/[name].md` | Per-character frame (genre-appropriate), role, story position | story-architect | prose-editor |
| `chapters/[nn]-[slug].md` | Planning artifact: scene goal, tension, turn, character state, continuity markers | story-architect | prose-editor |
| `chapters/[nn]-[slug]-review.md` | Editorial output: alignment report, flags, draft notes | prose-editor | writer |
| `handoff/handoff-context-v[N].json` | Point-in-time snapshot: gate status, artifact index, open questions | story-architect | prose-editor |

**Why separate files for story-architect and prose-editor output at the chapter level:** If both skills write to the same chapter file, the second writer's additions risk overwriting or corrupting the first's. Separate files with a `-review.md` suffix give prose-editor a dedicated output space that story-architect never touches.

**Cross-character relationships artifact:**
Character relationships are centralized in `cross-character-relationships.md` rather than scattered across individual character files. A relationship between A and B has one home — not two potentially inconsistent descriptions.

```
[Protagonist] — [Mentor]:
  type: inherited_obligation
  dynamic: [writer-provided]
  status: confirmed | [UNRESOLVED: dynamic not yet articulated]
```

**Exact field schemas** for all artifacts are in `references/artifact-schemas.md`. The SKILL.md loads this reference file when writing artifacts, not during general conversation.

---

### 4.3 — Memory Index Structure

Story-architect follows the project's existing memory conventions. It writes one memory file per active story project, named by topic rather than by file path.

**Memory file:** `memory/project_[project-slug].md`

**Memory file contents:**

```markdown
---
name: story-architect: [Project Title]
description: Active story-architect project — [genre_tag], last gate: [gate_name]
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

**MEMORY.md index entry:**

```
- [story-architect: Project Title](project_[slug].md) — [genre_tag], last gate: [gate]
```

**What the memory file does not contain:** story content of any kind, artifact file contents, session transcripts, summaries of planning decisions. It is a navigation pointer only.

**Memory file lifecycle:**
- Created at intake gate when a new project is started
- Updated at each gate clearance (`last_gate_cleared`, `last_session`)
- Updated when handoff is written (`status → handoff_written`)
- Not deleted when a project is complete — retained as a pointer for future reference

---

### 4.4 — Gate State Tracking

Gate state is tracked in `story-manifest.md` (the durable planning record) and mirrored at handoff time in `handoff-context.json` (the prose-editor intake snapshot).

**Gate log in story-manifest.md:**

```markdown
## Gate Log
premise_lock:         cleared: 2026-03-20
character_commitment: cleared: 2026-03-22
structure_approval:   in progress
chapter_direction:    not cleared
handoff:              not written
```

Gate status values: `not cleared` / `in progress` / `cleared: [date]` / `revised: [date] (originally cleared: [date])`

**The handoff sync rule:**
`handoff-context.json` is a **point-in-time snapshot** generated only at the handoff event. It is not kept incrementally in sync with `story-manifest.md`. Prose-editor treats it as accurate as of the `created_at` timestamp.

If the writer revises planning after the handoff was written, they must regenerate the handoff before prose-editor uses it. The skill surfaces this automatically: "Your handoff was written on [date] but you've revised [gate] since then. Regenerate the handoff before returning to prose-editor."

---

### 4.5 — Anti-Ghostwriting Enforcement

The anti-ghostwriting constraint is a behavioral rule enforced at the artifact write level.

**The behavioral rule:**
The skill is instructed never to write field content it generated, inferred, or synthesized from context. Before writing any field, the skill confirms the content originated from the writer. If the content fails this check, the field is not written. An `[UNRESOLVED]` annotation is written instead and a friction question fires.

**What "writer-provided" means:**
- Writer typed it directly in this session
- Writer confirmed it in response to a friction question (with genuine engagement — see hollow compliance below)
- Writer pasted it from pre-existing notes and the skill organized it without adding content

**What does not count as writer-provided:**
- Writer said "yes," "sure," or "that works" to a skill-generated suggestion
- Writer said "I don't know, you decide"
- Skill inferred it from context ("based on what you said earlier, I'm assuming...")

**Why passive agreement doesn't count:** Passive agreement to skill-generated content is the ghostwriting failure mode in a different form. A writer who says "yes" to a skill-generated premise hasn't written a premise — they've approved one the skill wrote.

**Hollow compliance detection:**
If a writer is answering every friction question in under ten words with no self-generated specificity, the skill names the pattern after three consecutive thin answers — not accusatorially, but observationally: "Your last few answers are technically complete but very brief. Would it help to take one of these deeper before we move on?" This is advisory only.

**Field states (three only):**
- **Writer-provided:** content traces to writer input and passes minimum-viable resolution
- **Unresolved:** field is empty, thin, or unanswered — contains `[UNRESOLVED: reason]`
- **Confirmed:** writer-provided AND explicitly confirmed during gate clearance — logged in gate_log

There is no "skill-generated" field state. That content is never written.

---

### 4.6 — Context Budget Management

Story-architect loads artifact files selectively to prevent context exhaustion across long projects.

**Tiered loading strategy:**

| Tier | What loads | When |
|---|---|---|
| Always | `story-manifest.md` | Every session, every turn — project identity and gate state |
| On gate entry | Gate's required-field reference from `references/artifact-schemas.md` | When that gate becomes active |
| On character work | `characters/[name].md` for the character being discussed | When writer mentions a specific character by name |
| On structure work | `structure-map.md`, `cross-character-relationships.md` | When Structure Approval gate is active or writer asks about structure |
| On chapter work | `chapters/[nn]-[slug].md` for the chapter being planned | When Chapter Direction gate is active for that chapter |
| On handoff | All artifact paths (index only, not full contents) | When generating `handoff-context.json` |
| Never during planning | `handoff-context.json`, `chapters/[nn]-[slug]-review.md` | These are write-only or prose-editor-facing artifacts |

**Token budget rule:** If the current session context exceeds 60% capacity with loaded artifacts, the skill surfaces a warning and asks the writer which artifacts to keep vs. unload. It does not silently drop files.

---

### 4.7 — KB Integration at Gates

Story-architect uses the existing `writing_mentor_kb` rather than re-implementing craft logic in SKILL.md. KB files are loaded at the gate where their content is most relevant.

| Gate | KB files loaded | Purpose |
|---|---|---|
| **Premise Lock** | `GUIDE.PREMISE.VIABILITY.md`, `CHK.PREMISE.VIABILITY.md` | Gate readiness criteria, checklist for required field validation |
| **Character Commitment** | `GUIDE.CHARACTER.WANT_NEED_MISBELIEF.md`, `GUIDE.CHARACTER.AGENCY_AND_DECISION_PRESSURE.md` | Character frame definition, agency validation |
| **Structure Approval** | `GUIDE.STRUCTURE.WEILAND_BEAT_MAP.md`, `GUIDE.STRUCTURE.TWO_HALVES_MAJOR_BEATS.md`, `GUIDE.STRUCTURE.MIDPOINT_SHIFT.md` | Structure framework options, key structural marker definitions |
| **Chapter Direction** | `CHK.SCENE.PURPOSE.md`, `MAP.CONTINUITY.LEDGER_TEMPLATE.md` | Scene goal validation, continuity marker template |

**Loading rule:** KB files for a gate are loaded when that gate becomes active, not before. They are unloaded when the gate closes.

**What story-architect does not do with the KB:** It does not surface KB content as craft education to the writer. If the writer asks a craft question, story-architect routes to writing-guide. Story-architect uses KB files internally to evaluate gate readiness.

---

### 4.8 — Anti-Drift Controls

Four mechanisms prevent planning decisions from drifting away from what the writer actually committed to.

**1. Annotation persistence**
Unresolved fields carry `[UNRESOLVED: reason]` annotations that persist until the field passes resolution. The skill re-adds annotations if the writer manually removes them without resolving the underlying gap.

**2. Gate clearance as commitment**
Gates require explicit writer confirmation. Commitment is logged with a date. A writer who wants to change a confirmed decision must go through the gate revision flow — which surfaces downstream consequences before any changes are made.

**3. Coherence alert — trigger rule**
Coherence alerts fire when new writer input contradicts a field that has been gate-cleared. They do not fire on unresolved or in-progress fields — conflict is expected there.

```
COHERENCE ALERT:
  New input contradicts a confirmed field.

  Confirmed at [gate] on [date]: [current field value]
  New input implies: [what changed]

  Which is current? Revise the confirmed field via gate revision,
  or clarify that the new input was exploratory?
```

**4. Genre tag change re-evaluation**
If the writer changes their genre tag, the skill re-runs the gap scan against the new genre's required-field set before accepting the change. Fields confirmed under the old genre are re-evaluated against the new genre's requirements and re-annotated with their mapping status.

---

### 4.9 — Writer Commands

Two read-only commands available at any point during a session:

**`plan summary`** — renders the current state of `story-manifest.md` in human-readable form, organized by confirmed vs. open.

**`gap report`** — surfaces all `[UNRESOLVED]` annotations across all active artifacts in one view, organized by artifact and severity.

Both commands are conversation-only — they do not write to any artifact.

---

### 4.10 — Diagnostic Intervention

Story-architect has a fourth intervention type beyond friction questions, gap flags, and coherence alerts: the **diagnostic intervention**. This fires when the skill detects a structural problem that no amount of field-filling will resolve — a premise-level contradiction that invalidates the story's basic logic.

**Diagnostic response format:**

```
DIAGNOSTIC — Structural Issue Detected

This isn't a gap in a field — it's a structural contradiction that
friction questions alone can't resolve.

What I'm seeing:
  [Plain-language description of the structural problem]

Why this matters:
  [What breaks downstream if this isn't addressed]

Options:

  1. Talk through the contradiction — explore what a restructured
     premise might look like (Exploratory mode, sprint format)
  2. Revise the affected gate — return to [gate] and reconsider the
     decisions that created this contradiction
  3. Continue anyway — note the contradiction and proceed; address
     it during drafting (Exploratory mode only)
```

The diagnostic intervention does not suggest a fix. It names the problem, identifies the scope, and offers a path — the writer decides how to proceed.

---

### 4.11 — Multi-Gap Answer Processing

When a writer's response addresses multiple open fields simultaneously, story-architect processes all of them — not just the one it asked about.

**Processing rule:** After receiving a writer response, the skill runs the gap scan on all currently open fields against the new content. Any field that passes minimum-viable resolution is updated and its annotation removed. The skill acknowledges what was resolved before firing the next friction question:

```
That resolved three things at once:
  ✓ core_tension — updated
  ✓ protagonist_stake — updated
  ✓ inciting_premise — updated

One remaining gap before Premise Lock is ready:
  thematic_question — [friction question fires here]
```

**Why process all at once:** Forcing the writer to re-state content they've already provided wastes session time and treats the writer as a form-filler rather than a thinker.

---

### 4.12 — Special Protocols

**Real-person disclosure for memoir:**
At the Character Commitment gate, for any character whose `role` field indicates a real person (memoir/CNF genre tags only), the skill surfaces a one-time advisory asking the writer to consider disclosure, attribution, and differing perspectives. This is advisory only — never a gate hold.

**`handoff_notes` scope constraint:**
The `handoff_notes` field in `handoff-context.json` is limited to factual metadata only — what gates were cleared, what changed since the prior handoff. It does not contain interpretive narrative or skill-generated characterizations of the writer's decisions.

---

## Section 5 — Integrated Workflow Design

---

### 5.1 Architectural Model: Layered Ecosystem

The three modules are a layered ecosystem with a hub-and-specialist architecture. They are not interchangeable modes, isolated tools, or a monolithic application.

**writing-guide** is the persistent hub — always available, handling general craft guidance, mentorship, and process support across the full writing lifecycle. It does not specialize in story structure or prose consistency; it handles breadth.

**story-architect** and **prose-editor** are specialist skills invoked when the writer's need exceeds the hub's scope. They handle depth within specific domains.

Each module has a single clear purpose, communicates through defined interfaces, and can be understood and tested independently. The writer always starts at writing-guide. Specialist invocations are offered, not forced — the writer can decline any invocation and remain in the hub layer. Transitions between modules are writer-triggered, not automatic.

---

### 5.2 Entry Points: When the Writer Enters Each Workflow

**Entry into writing-guide (always-on hub):**
- The writer opens a session with any writing question, process check, or project touch
- writing-guide checks for a live `story-architect-session.json` and loads relevant state if present
- General guidance, craft reminders, checkpoint prompts, and structural hygiene happen here
- No special invocation required

**Entry into story-architect (three triggers):**

1. **Explicit invocation** — the writer asks to start a new project, develop an existing idea, or work through a structural problem. writing-guide offers story-architect; the writer confirms.

2. **Toll gate escalation** — writing-guide reaches a toll gate and the writer's project has no viable premise on record. writing-guide surfaces the gap and offers to invoke story-architect.

3. **Session resume** — the writer returns to an in-progress story-architect session. writing-guide detects the open session artifact and asks if the writer wants to continue from the last completed gate.

**Entry into prose-editor (three triggers):**

1. **Explicit invocation** — the writer pastes a chapter or passage and requests consistency review, continuity check, or tone analysis.

2. **Post-handoff invocation** — story-architect generates `handoff-context.json` at Chapter Direction gate. writing-guide informs the writer that the structural foundation is complete and offers to invoke prose-editor.

3. **Diagnostic escalation** — writing-guide or story-architect identifies a prose-level problem and escalates to prose-editor with the problem framed in the escalation payload.

---

### 5.3 System Transitions: Ideation → Development → Prose Review

**Phase 1 — Ideation (writing-guide)**

The writer brings a raw idea. writing-guide handles this through process guidance, clarifying questions, and craft mentorship. No structural planning artifacts are created yet.

*Writer can enter story-architect when:* they want to move from "thinking about" to "building the structure." writing-guide offers story-architect and waits for the writer's confirmation.

**Phase 2 — Story Development (story-architect)**

The writer works through the four planning gates. Writer answers are the only permitted content in planning fields.

*Writer can enter prose-editor when:* all four planning gates are passed and `handoff-context.json` has been generated. The writer can also decline and draft independently before invoking prose-editor later.

**Phase 3 — Prose Review (prose-editor)**

The writer drafts chapters and brings them to prose-editor for review. prose-editor outputs a review file and routes problems to the appropriate layer.

**Return loops — non-linear re-entry:**

When prose-editor escalates a structural contradiction to writing-guide, writing-guide presents the writer with a concrete decision:

> **Structural conflict detected** (example framing):
> prose-editor found that [character] acts in chapter 7 in a way that contradicts the committed arc in `character-profiles.md`. You have three paths:
> - **Revise the plan** — return to story-architect at Character Commitment gate and update the arc
> - **Revise the prose** — the plan is right; the chapter drifted. Return to the chapter with the committed arc in view
> - **Defer** — flag this conflict and continue drafting; resolve before the next prose-editor review

The writer chooses. If the writer chooses "Revise the plan," writing-guide invokes story-architect at the relevant gate, carrying the prose-editor escalation payload as context.

---

### 5.4 Inter-Module Handoffs: How Outputs Become Inputs

**story-architect → prose-editor**

The primary handoff artifact is `handoff-context.json` (versioned in `handoff/` subdirectory). This is a point-in-time snapshot. If the writer returns to story-architect and makes structural revisions, story-architect generates a new versioned handoff. When prose-editor next loads its context, it detects the version mismatch and asks the writer to confirm which version to use before continuing.

**story-architect → writing-guide**

story-architect writes session state to `story-architect-session.json`. writing-guide reads this at session start to determine whether an active project exists, which gate was last completed, and how to contextualize current questions.

**prose-editor → writing-guide**

When prose-editor flags a problem exceeding its scope, it produces an escalation payload that writing-guide uses to triage: route to KB resources, offer story-architect re-entry, or activate anti-ghostwriting enforcement.

**prose-editor → story-architect**

prose-editor does not write to story-architect's planning artifacts directly. When a review reveals that a structural artifact needs revision, prose-editor surfaces the conflict with enough context for the writer to make the plan-vs-prose decision. The writer decides; story-architect executes only on writer instruction.

---

### 5.5 Escalation Types and Routing

All inter-skill communication uses the skill-to-skill JSON API defined in writing-guide. The full schema is in `references/artifact-schemas.md`.

| Type | Source | Routes to | Severity | Writer action required |
|---|---|---|---|---|
| `structural-gap` | any | story-architect | flag | Resolve missing field at appropriate gate |
| `voice-drift` | prose-editor | writing-guide | advisory | Review KB guidance; revise if writer agrees |
| `continuity-contradiction` | prose-editor | writing-guide → story-architect | hold | Writer chooses plan-revise or prose-revise path |
| `gate-hold` | story-architect | story-architect (held in place) | hold | Provide required field inputs to pass gate |
| `diagnostic` | story-architect / writing-guide | writing-guide (Diagnostic mode) | hold | Premise-level structural contradiction requiring foundational rework |
| `ghostwriting-risk` | any | enforced locally, not routed | non-bypassable | N/A — skill refuses the request in place |

**On ghostwriting-risk:** Each module enforces its own anti-ghostwriting constraint locally. A `ghostwriting-risk` escalation is not routed to writing-guide for central adjudication — it is handled at the point of detection. No escalation pathway exists for this type because routing implies the possibility of override; there is none.

---

### 5.6 How writing-guide Interacts with the Specialist Modules

**As a router:** writing-guide reads session state and guides the writer toward the right tool for their current need. It does not attempt to provide story structure guidance that story-architect should handle, or prose review that prose-editor should handle. This boundary exists because diluting specialist depth into the hub layer produces worse outcomes than routing.

**As a persistent context layer:** writing-guide holds the writer's general project context across sessions. It uses this to frame KB loading and specialist invocations.

**As an escalation triage layer:** When specialist modules surface problems, writing-guide interprets the escalation payload, routes to the right resource or module, and handles the writer-facing communication in a consistent voice.

**As a craft mentor during drafting:** Between story-architect and prose-editor sessions, writing-guide remains available for craft questions and process checks without requiring a formal review invocation.

---

### 5.7 Workflow Coherence Principles

1. **Single entry surface.** The writer always starts at writing-guide. Specialist invocations are offered, not forced. *Why: forcing writers through a specialist intake on every session creates friction that erodes trust. The hub layer should feel like a writing conversation, not a routing menu.*

2. **Artifacts are ground truth.** A module's behavior is determined by what is in the writer's artifact files, not what was said in a prior session. *Why: conversation history drifts and is not inspectable. File state is deterministic, versionable, and testable.*

3. **Writer decisions cannot be inferred or filled.** Any planning field requiring a writer decision that is not present in an artifact is treated as unresolved. No module fills planning fields on the writer's behalf. *Why: inferred decisions are the primary mechanism by which writing-support tools drift into ghostwriting. The constraint is structural, not behavioral — it prevents the pathway from existing.*

4. **Structural rework returns to source.** If prose-editor identifies a structural problem, it does not attempt to resolve it at the prose layer. It routes back to story-architect at the relevant gate. *Why: resolving structural problems at the prose layer creates surface-level fixes that mask foundational contradictions — the problem returns in a later chapter.*

5. **Transitions are offered, not triggered.** No module automatically advances the writer from one phase to another. Transition conditions create an offer; the writer accepts or defers. *Why: writers do not work in linear phases. Automatic advancement removes writer agency at the moments it matters most.*

---

### 5.8 Lifecycle Trace: From Premise to First Chapter Review

1. Writer opens a session with a rough premise idea. writing-guide handles it as general craft. No artifacts created yet.
2. Writer says "I want to structure this properly." writing-guide detects no active story-architect session, offers invocation. Writer confirms.
3. Writer works through Premise Lock and Character Commitment gates. Structure Approval stalls — midpoint shift is unresolved. Gate holds. Writer provides the answer. Gate passes.
4. Writer completes Chapter Direction gate. story-architect generates `handoff-context-v1.json`. writing-guide informs the writer: structural foundation is complete, prose-editor is available when the first chapter is ready.
5. Writer drafts chapter 1 independently. Pauses to ask writing-guide a craft question about pacing. writing-guide answers using KB resources — no specialist invocation needed.
6. Writer brings chapter 1 to prose-editor. prose-editor loads `handoff-context-v1.json`, runs review pass. Flags a continuity issue. Severity: hold.
7. Escalation routes to writing-guide. writing-guide presents the plan-vs-prose decision. Writer chooses "revise the plan." writing-guide invokes story-architect at Character Commitment gate.
8. Writer updates the character commitment. story-architect regenerates `handoff-context-v2.json`. writing-guide flags version mismatch. Writer confirms to use v2.
9. Prose-editor review resumes against v2. No further holds. Review file written to `chapters/01-chapter-one-review.md`.

---

## Section 6 — Initial Implementation Model

---

### 6.1 Skill File Structure

The ecosystem is three Claude skills. writing-guide already exists. story-architect and prose-editor are new builds. Each skill follows three-level progressive disclosure: metadata (always in context), SKILL.md body (loaded on invocation), and references/ (loaded on demand). SKILL.md files stay under 500 lines — when a feature requires more than a few lines to specify, it belongs in references/ with a pointer and load condition in SKILL.md.

**writing-guide** (existing — extend only):
```
~/.claude/skills/writing-guide/
├── SKILL.md                          ← existing, ~625 lines, do not modify
└── references/
    ├── specialist-routing-guide.md   ← new: when to offer story-architect vs. prose-editor
    └── escalation-triage.md          ← new: runtime reference for escalation type routing
```

**story-architect** (new):
```
~/.claude/skills/story-architect/
├── SKILL.md                          ← main skill, ~500 lines
└── references/
    ├── artifact-schemas.md           ← all field schemas, handoff-context schema
    ├── genre-vocabulary.md           ← structural vocabulary by genre tag
    ├── character-frames.md           ← all four genre character frames in full
    ├── friction-patterns.md          ← hollow compliance detection, gap scan logic
    └── kb-gate-map.md                ← which KB files load at which gates
```

**prose-editor** (new):
```
~/.claude/skills/prose-editor/
├── SKILL.md                          ← main skill, ~400 lines
└── references/
    ├── review-protocol.md            ← the five-pass review logic
    ├── escalation-payloads.md        ← escalation type definitions and routing rules
    └── voice-anchor-guide.md         ← how to apply voice anchors for consistency checking
```

---

### 6.2 Artifact Taxonomy

All writer-owned artifacts live under `docs/writing/[project-slug]/`.

**Session state** (transient — not committed to git):

Session state tracks live context: active gate, project slug, session mode, context budget. It should be fully reconstructible from the planning artifacts and is not a source of truth. Committing it would create noise in version history without adding recoverability.

| File | Purpose |
|---|---|
| `story-architect-session.json` | Active gate, project slug, session mode, context budget state |

**Project planning artifacts** (persistent — writer-owned, version-controlled):

| File | Created at gate | Contains |
|---|---|---|
| `story-manifest.md` | Intake (stub), updated at each gate | Logline, premise, genre tag, core conflict, thematic argument, tone markers, gate log |
| `structure-map.md` | Structure Approval | Act/part breakdown, major turning points, pacing notes |
| `character-profiles.md` (per character) | Character Commitment | Character frame (genre-appropriate), key relationships, voice anchors |
| `cross-character-relationships.md` | Character Commitment | Cross-character relationship map |
| `chapter-map.md` (per chapter) | Chapter Direction | Chapter-level goals, POV assignments, escalation notes |
| `continuity-ledger.md` | Chapter Direction (initial) | Running log of committed facts; prose-editor flags additions, writer confirms before write |

**Handoff artifacts** (versioned snapshots):

```
docs/writing/[project-slug]/handoff/
├── handoff-context-v1.json
├── handoff-context-v2.json
└── ...
```

Each version is a point-in-time snapshot. Prior versions are retained. prose-editor detects version mismatch by comparing the version field in its loaded context against the latest file in `handoff/` and flags it before starting a review.

**Review artifacts** (per-chapter outputs):

| File | Created when | Contains |
|---|---|---|
| `chapters/[nn]-[slug]-review.md` | Each prose-editor review pass | Flagged issues by severity, writer-directed next actions, pass/hold status |

**Memory index entry:**

| File | Location | Contains |
|---|---|---|
| `project_[slug].md` | `memory/` | Project title, genre tag, active gate, artifact paths, last session date |

---

### 6.3 Gate and Checkpoint Structure

**story-architect — four planning gates:**

| Gate | Held until | Key fields required | KB files loaded |
|---|---|---|---|
| Premise Lock | Premise is viable and writer-confirmed | Genre tag, premise statement, conflict anchor | GUIDE.PREMISE.VIABILITY, CHK.PREMISE.VIABILITY |
| Character Commitment | Protagonist frame fully populated | Want/need/misbelief (or genre equivalent), antagonist logic, voice anchors | GUIDE.CHARACTER.WANT_NEED_MISBELIEF, GUIDE.CHARACTER.AGENCY |
| Structure Approval | Act skeleton writer-confirmed | Act 1/2/3 pivots, midpoint shift, turning point logic | GUIDE.STRUCTURE.WEILAND_BEAT_MAP, GUIDE.STRUCTURE.MIDPOINT_SHIFT |
| Chapter Direction | Chapter intent map complete | Chapter-level goals, POV assignments, escalation notes | CHK.SCENE.PURPOSE, MAP.CONTINUITY.LEDGER_TEMPLATE |

Voice anchors are collected at Character Commitment rather than Premise Lock. Voice is partly a function of whose perspective the reader occupies and how that character perceives the world — it becomes specifiable once the protagonist frame is established.

Gate holds are non-negotiable in Rigorous and Workshop modes. The distinction between them operates at the friction layer, not the gate layer — Rigorous asks "is this field resolved?"; Workshop asks "does this field hold under contradiction?"

In Exploratory mode, the writer can pass a gate with unresolved fields by explicitly acknowledging them as deferred. Deferred fields resurface at the next session and cannot be marked `confirmed` while still deferred.

**Session mode checkpoint (soft gate — fires at session start):**

For returning sessions: the prior mode is loaded and shown for confirmation or change.
For new sessions: the default is **Exploratory**. The skill presents all three modes briefly; Exploratory is pre-selected.

**prose-editor — four review checkpoints:**

| Checkpoint | Description | Blocking? |
|---|---|---|
| Intake check | Verify handoff context is present and version-current | Conditional — see below |
| Review pass | Five-pass analysis: premise alignment, character consistency, voice coherence, continuity ledger, structural positioning | — |
| Issue triage | Categorize issues as advisory / flag / hold | Holds block continuation; advisories do not |
| Writer resolution | Writer acknowledges review, states which holds they will address | Yes — holds must be acknowledged before review file closes |

**Prose-editor without a story-architect handoff:**

When invoked without a handoff, prose-editor operates in **context-only mode**: the writer provides a minimal context block at invocation (genre tag, protagonist name or subject, one-sentence premise, voice notes). prose-editor runs a reduced review pass — voice coherence, continuity ledger, and character consistency checks remain active; premise alignment and structural positioning checks are suspended. Context-only mode is declared in the review file header.

---

### 6.4 Review Loops

**story-architect — continuous gap scan loop:**

After every writer response, the skill runs a 3-pass gap scan. Gaps from passes 1 and 2 queue as friction questions for the skill's next response — they do not interrupt the writer's current response. Gaps from pass 3 become holds. The writer can request a `gap report` at any time.

**prose-editor — scoped second-pass loop:**

When a review produces holds:
1. Writer addresses holds in their prose
2. Writer re-submits the revised chapter
3. Prose-editor runs a scoped second pass — checks only previously held items
4. If all holds resolve, the review closes

If unresolved holds remain after the second pass, prose-editor generates a `diagnostic` escalation to writing-guide. writing-guide surfaces the conflict and offers story-architect invocation at the gate most relevant to the unresolved issue. Further prose iteration at this point would produce surface fixes that mask a foundational problem.

**Cross-module review loop (structural revision mid-draft):**

1. Writer re-enters story-architect at the relevant gate
2. story-architect runs gate assessment after writer updates the artifact
3. story-architect generates `handoff-context-v[N+1].json`
4. On next prose-editor invocation, the skill compares its loaded context version against the latest file in `handoff/` — if they differ, it flags the mismatch and asks the writer to confirm
5. Scoped second pass resumes against the confirmed handoff version

---

### 6.5 Author-Control Safeguards

**Field provenance:**

All planning artifact fields carry one of three provenance states:
- `writer-provided` — content originated from writer input
- `unresolved` — field is required but not yet populated
- `confirmed` — writer has explicitly acknowledged this field as complete at gate passage

`confirmed` status is assigned at gate passage as a batch operation — consistent with multi-gap answer processing (Section 4.11). It cannot be inferred from conversational context.

**Advisory / hold / deferred distinction:**
- **Advisory** — issue flagged; writer can proceed. Logged in review file.
- **Hold** — gate or review will not advance until the writer resolves the issue. Cannot be overridden.
- **Deferred** (Exploratory mode only) — writer explicitly labels an unresolved required field as deferred. Surfaces at next session.

**No artifact writes without explicit instruction:**

No module writes to or overwrites a planning artifact without explicit writer instruction. The writer can ask the skill to surface what it would flag without producing any artifact output — this follows naturally from the no-write constraint, not as a separate mode.

**Anti-ghostwriting constraint (distributed):**

Each module enforces this constraint locally. The permitted boundary for prose-editor: editing support is permitted only on writer-provided text. The skill may flag, reframe, or suggest structural alternatives for text the writer has submitted. It may not generate new sentences, events, dialogue, or character decisions not present in the submitted prose.

---

### 6.6 Genre and Style Flexibility

**Genre abstraction layer:**

The system supports four genre tags: `narrative-fiction`, `memoir`, `creative-nonfiction`, `long-form-essay`. Set at Premise Lock, stored in `story-manifest.md`. Controls structural vocabulary, character frame, KB file selection, and gate question language.

**Voice anchors:**

Writer-defined descriptors collected at Character Commitment gate. Stored in `character-profiles.md` and loaded by prose-editor as its consistency reference. The system does not impose or suggest a voice; it enforces consistency with the voice the writer has declared.

**Mixed-genre projects:**

Writer sets a primary genre tag and notes secondary characteristics in `story-manifest.md`. Structural vocabulary and gate language adapt to the primary tag. At any gate, the writer can request KB resources from a secondary genre tag — writer-initiated override only.

**Project-type flexibility:**

| Project type | Artifact notes |
|---|---|
| Short story / standalone essay | Single-chapter map; continuity ledger optional; prose-editor operates without chapter sequence |
| Novel / book-length memoir | Full chapter map; continuity ledger active; handoff versioning relevant |
| Series / multi-book | Outside current scope. The per-project path allows future nesting under a series root if needed. |

---

## Section 7 — Risks, Tensions, and Open Design Questions

---

### 7.1 Overview

This section documents where the design has made choices involving real trade-offs, where known risks exist that the design does not fully resolve, and where decisions have been deliberately deferred. The three categories — design tensions, implementation risks, and open questions — are cross-referenced where one feeds the other.

---

### 7.2 Design Tensions

**Tension 1: Productive friction vs. friction fatigue**

The entire value proposition of story-architect rests on friction — challenging weak ideas, holding gates, requiring rigorous answers before advancing. Friction that fires at the right moment on a genuinely underdeveloped idea is useful. Friction that fires on a writer who knows what they're doing, or who is early in an exploratory phase, is obstruction.

The design's resolution is session modes: Exploratory, Rigorous, Workshop. The writer dials friction up or down. But this creates a secondary risk — writers default to Exploratory and never experience the gates that would actually develop their work. A writer who tries Workshop mode once, finds it grueling, and switches permanently to Exploratory has effectively opted out of the feature that makes the tool distinctive. The design has no model for how trust in higher-friction modes gets built over time.

→ *Deferred to OQ4: Onboarding model for session mode selection.*

---

**Tension 2: Structure-first assumption vs. discovery writers**

The story-architect model assumes structure precedes prose: commit the premise, develop the character frame, approve the act skeleton, map the chapters — then draft. This is the right model for writers who have ideas and need help structuring them. It is the wrong model for discovery writers, who find their characters through scenes and cannot commit an act skeleton before they have written into the story.

The gate sequence is structure-first at its core. A discovery writer using this tool in Exploratory mode with aggressive deferrals might extract value from the friction questions as reflective prompts rather than blocking requirements — but the design has not been built for that use case. This is a meaningful exclusion. Future implementers should not assume coverage they do not have.

→ *Deferred to OQ8: Discovery writer gate model adaptation.*

---

**Tension 3: Consistency enforcement vs. intentional voice evolution**

Prose-editor's core value is detecting voice drift across chapters. But a chapter that sounds different from the others might be intentional — a writer may deliberately modulate tone at a structural inflection point. The system has no mechanism to distinguish intentional voice modulation from unintended drift.

The current design flags voice differences against the declared voice anchors and surfaces them as advisories. If the writer consistently overrides voice-drift advisories because their choices are intentional, the advisories become noise and eventually stop being read. A mechanism for declaring intentional deviations would resolve this, but the design details are not resolved in this spec.

→ *Deferred to OQ2: Intentional voice deviation protocol.*

---

**Tension 4: Anti-ghostwriting constraint vs. writer generative crises**

This tension only applies to one specific failure mode: a writer who has no ideas at all — not a structural block, but a genuine generative crisis. The two scenarios are different:

- **Structural block** — the writer has ideas but cannot develop them. writing-guide's friction questions already address this. This is within scope.
- **Generative crisis** — the writer has nothing to react to, no starting material, no direction. They push the tool toward content generation not as a shortcut but out of genuine need. The anti-ghostwriting constraint fires. The skill refuses. The writer is still stuck. The tool has no escalation path for this scenario.

The design is built for writers with ideas who need structural support, not for writers in generative crises.

→ *Deferred to OQ5: Stuck writer intervention type in writing-guide.*

---

**Tension 5: The editing support boundary under indirect framing**

Section 6.5 defines the permitted boundary: editing support is permitted only on writer-provided text; the skill may not generate new sentences, events, or character decisions not present in the submitted prose.

This boundary is clear when requests are explicit. It becomes soft under good-faith indirect framing: "the dialogue doesn't land — what would make it work?" "this sentence is too passive — how would you fix it?" "the character wouldn't say this — what would she say instead?" None of these are ghostwriting requests in intent. All of them, if answered prescriptively, produce content the writer did not write.

The anti-ghostwriting constraint is described as non-bypassable, but the detection of boundary violations is behavioral. Getting this calibration right is one of the hardest implementation challenges in the prose-editor module. The current spec does not provide sufficient behavioral guidance for it.

→ *Deferred to OQ9: Editing support boundary specification for prose-editor.*

---

### 7.3 Implementation Risks

**Implementation Prerequisite: KB coverage audit**

The existing `writing_mentor_kb` was built primarily for fiction. The genre abstraction layer assumes KB files exist at equivalent depth for memoir, creative-nonfiction, and essay. This is a known condition, not an uncertain risk — the non-fiction genre coverage is thinner. The `kb-gate-map.md` referenced in Section 6.1 does not yet exist and will need to be built.

Before story-architect is implemented, the KB must be audited against the four genre tags and the four gates to identify which gate-genre combinations have adequate coverage and which need new KB documents. This is a hard prerequisite, not a post-build improvement.

---

**Risk 1: Artifact complexity overwhelming low-investment writers**

The number of gate questions required to populate the full artifact set may feel like overhead for a writer doing a short story or standalone essay. The design notes project-type flexibility but does not specify a minimal artifact set for low-investment sessions.

→ *The deferred decision is in OQ3: Minimal viable artifact set for short-form projects.*

---

**Risk 2: Context window exhaustion mid-session**

The tiered context loading strategy and 60% capacity threshold warning were designed to manage context budget. But the spec does not define what happens after the warning — does the session save state and continue in a new session? Is there compaction? Does the writer lose in-progress gate work?

A story-architect session working through multiple gates with an active KB load could realistically hit context limits before completing Chapter Direction.

→ *Deferred to OQ7: Session recovery behavior under context pressure.*

---

**Risk 3: Long-hiatus session drift**

A writer who returns after a long absence may find their committed planning artifacts no longer match their evolved mental model of the project. The system enforces consistency with the artifacts, not with the writer's current intuition.

→ *Deferred to OQ10: Re-orientation protocol for long-hiatus session resume.*

---

### 7.4 Open Design Questions

*Decisions deferred from this spec that require resolution before story-architect is built.*

**OQ1: Structure Approval gate for non-linear and experimental projects**
The Structure Approval gate assumes a project has an act skeleton. Experimental fiction, fragmented memoir, and anti-narrative essay collections may explicitly reject linear structure. Should the gate adapt its required fields based on the writer's declared form? *(Related: Tension 2)*

**OQ2: Intentional voice deviation protocol**
Should writers be able to annotate a chapter with an intentional-deviation flag that tells prose-editor to treat voice differences as confirmed rather than flagged? If yes, where does the annotation live? If no, what prevents voice-drift advisories from becoming noise? *(Related: Tension 3)*

**OQ3: Minimal viable artifact set for short-form projects**
What is the minimum artifact footprint for a short story or standalone essay session that still provides structural value? Can Character Commitment and Structure Approval gates be combined into a single lightweight pass for short-form work? *(Related: Risk 1)*

**OQ4: Onboarding model for session mode selection**
New users will default to Exploratory. The design has no model for how a writer learns when Rigorous or Workshop mode is appropriate, or what a Workshop session actually feels like before they have experienced one. *(Related: Tension 1)*

**OQ5: Stuck writer intervention type in writing-guide**
A dedicated intervention type for writers in genuine generative crises — one that asks questions calibrated to produce raw material the writer can react to, without generating content — would extend the tool's coverage. Should this be designed into writing-guide as a distinct intervention type? *(Related: Tension 4)*

**OQ6: Prose-editor context-only mode — quality threshold**
Is context-only mode genuinely useful, or does it produce review shallow enough to mislead the writer about the tool's capabilities? This should be validated before prose-editor is built.

**OQ7: Session recovery behavior under context pressure**
When story-architect approaches context window limits mid-session, what is the recovery behavior? Does the session save committed artifact state and continue in a new session? What does the writer see, and what gate work — if any — is lost? *(Related: Risk 2)*

**OQ8: Discovery writer gate model adaptation**
Can the gate model be adapted for discovery writers who develop structure through prose? Options include: a dedicated discovery mode with all gates optional, a post-draft structural review pass that inverts the gate sequence, or an explicit acknowledgment that discovery writers are out of scope for story-architect. *(Related: Tension 2)*

**OQ9: Editing support boundary specification for prose-editor**
What behavioral rules govern prose-editor's response to requests like "what would make this dialogue work?" or "what would this character say instead?" A concrete decision tree or set of redirect patterns is needed before prose-editor is implemented. *(Related: Tension 5)*

**OQ10: Re-orientation protocol for long-hiatus session resume**
Should story-architect offer a structured re-orientation pass when resuming after a significant gap — a review of committed artifacts before continuing gate work? If yes, what triggers it and how does it handle the case where the writer's current intuition conflicts with a previously confirmed field? *(Related: Risk 3)*

---

### 7.5 Out of Scope

*Entire design areas for future sessions — not deferred decisions within this spec.*

- **prose-editor full design** — this spec establishes prose-editor's role, integration points, and constraints; its internal design requires its own design session
- **writing-guide extensions** — the `specialist-routing-guide.md` and `escalation-triage.md` references added in Section 6.1 are specified by interface only
- **KB gap remediation** — the KB audit identified as a prerequisite in 7.3 is a precondition for story-architect implementation, not part of this spec
- **Series / multi-book projects** — the per-project artifact path allows future extension but cross-book continuity tracking is not designed into the current model
