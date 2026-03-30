# Writing Tool — prose-editor Design Spec
updated: 2026-03-29
scope: prose-editor module (second specialist skill)
status: complete — all 7 sections approved

---

## Section 1 — Executive Summary

The Writing Tool is a three-module, author-led writing support ecosystem built on Claude skills. This document specs the second specialist module: **prose-editor**.

**System overview:**
- **writing-guide** (existing, foundational): craft mentor, best-practices guide, process guardian. Primary entry point and escalation hub.
- **story-architect** (existing, first specialist): structured story-creation workflow for writers building a story from scratch. A planning coach and story architect — not a co-author.
- **prose-editor** (this spec): editorial review and continuity layer for writers actively drafting or revising. A diagnostic instrument — not a rewriter.

**Confirmed design decisions:**
- **Anti-ghostwriting:** Three compliant edit modes (flag-only, flag-with-diagnostic, flag-with-structural-alternative). No prose generation. Non-bypassable.
- **Staged build:** Phase 1 (editorial core: voice + prose quality + read-only continuity). Phase 2 (structural integration: alignment + character consistency + full continuity).
- **Checkpoint model:** Ordered phases (intake, review, triage, resolution), not sequential gates with field-level gating.
- **Dual severity axes:** Severity (Hold/Flag/Advisory) describes the issue. Intensity (gentle/standard/firm/diagnostic) controls delivery.
- **Integration format:** Prose Editing Toolkit modules adapted and consolidated into reference files. Detection patterns preserved, structural logic adapted to Writing Tool architecture.
- **Voice Profile:** Both creation paths available (story-architect stub at Character Commitment + prose-derived at first prose-editor run). Project-level artifact at `docs/writing/[project]/voice-profile.md`.
- **Pass ordering:** Recommended sequence with dispatch-driven overrides based on writer's stated concern.

**Core principle:** The writer is the editor. prose-editor is the diagnostic lens — it marks where the seams show, names why, and points toward the fix. It never holds the red pen.

---

## Section 2 — Use Case Breakdown: prose-editor

### Primary Purpose

Help writers evaluate and improve their drafted prose through structured diagnostic review. The module analyzes submitted text against voice consistency, prose quality patterns, and (when available) planning artifacts, producing actionable findings organized by severity. It makes writers see what their eye skips — AI-isms, rhythm monotony, diction weakness, echo patterns, pacing problems — without imposing a "correct" style.

### Ideal User Scenarios

- "I finished a chapter draft — does it sound like me or like AI?"
- "This section feels flat but I can't figure out why"
- "I revised with AI help — did it strip my voice?"
- "I need a fresh pair of eyes on this before I send it"
- "story-architect says my chapter plan is ready — now review my actual prose"
- "I just want a quick polish — flag the worst 5 things"

### Expected Inputs

- Writer-submitted prose (chapter, scene, section, or full draft)
- **Entry mode:** context-only (no planning artifacts) OR post-handoff (with story-architect artifacts)
- **Concern tag** selected at intake: `sounds-too-ai` / `needs-tightening` / `rhythm-off` / `full-review` / `quick-polish`
- **Edit mode** selected at intake: `flag-only` (default) / `flag-with-diagnostic` / `flag-with-structural-alternative`
- **Intensity** inherited from session or selected: gentle / standard (default) / firm / diagnostic
- Optional: Voice Profile (if exists), genre tag, audience

### Entry Points (4)

1. **Explicit invocation** — Writer pastes prose and requests review. writing-guide detects prose-review need and offers prose-editor.
2. **Post-handoff invocation** — story-architect generates `handoff-context.json`. Writer enters prose-editor with full planning context.
3. **Diagnostic escalation** — writing-guide or story-architect surfaces a prose-level problem (voice drift, rhythm issue) that exceeds craft guidance scope.
4. **Session resume** — writing-guide detects incomplete review artifact (`[nn]-[slug]-review.md` with `last_completed_pass` < total passes) and offers to continue from last completed pass.

### In-Session Interventions

| Intervention | Fires When |
|---|---|
| **Intake Prompt** | Session start — collects concern tag, edit mode, genre/audience if no handoff |
| **Voice Profile Offer** | No voice profile exists — offer to derive one from submitted text |
| **Review Pass Finding** | Each flagged issue during review execution |
| **Hold Escalation** | Hold-severity finding requires writer acknowledgment before review closes |
| **Per-Pass Checkpoint** | Between review passes — check if findings warrant continuing to next pass |
| **Continuity Fact Candidate** | Potential new fact identified in prose (Phase 1: flagged only; Phase 2: offer to write) |
| **Structural Conflict Alert** | Prose contradicts locked planning artifact (Phase 2 only) |
| **Review Report** | All passes complete — consolidated findings |
| **Iteration Gate** | Writer returns with revised text — choose re-run, verify, or start fresh |
| **Over-Editing Warning** | Third revision pass — warn of diminishing returns |

### Persistent Artifacts

| Artifact | Location | Written When |
|---|---|---|
| `voice-profile.md` | `docs/writing/[project]/` | First review (Path B) or Character Commitment gate (Path A); updated on writer confirmation |
| `[nn]-[slug]-review.md` | `docs/writing/[project]/chapters/` | Review pass complete |
| `continuity-ledger.md` | `docs/writing/[project]/` | Phase 1: read-only. Phase 2: after writer confirms candidate facts |

### Concern-to-Pass Dispatch Map

| Concern Tag | Passes Run | Pass Order |
|---|---|---|
| `sounds-too-ai` | Prose Quality (AI-ism focus), Diction Audit, Voice Coherence | 5, 3 |
| `needs-tightening` | Prose Quality (diction focus), Cliche Scan, Echo Detection | 5 |
| `rhythm-off` | Prose Quality (rhythm focus), Echo Detection, Readability/Pacing | 5 |
| `full-review` | All five passes | 1, 2, 3, 4, 5 (recommended) |
| `quick-polish` | Prose Quality (combined lint, compact), top 5 findings only | 5 |

**Phase note:** Passes 1 (structural alignment) and 2 (character consistency) only run when planning artifacts are available AND Phase 2 is implemented. In Phase 1 or context-only mode, these passes are skipped with a declaration in the review artifact.

---

## Section 3 — Key Features: prose-editor

### Feature 1 — The Five-Pass Review Protocol

**Why five passes:** Each pass examines prose through a different lens. Running all five ensures comprehensive coverage; running a subset (via dispatch) keeps reviews focused when the writer already knows their concern.

**Pass sequence (recommended ordering):**

| Pass | Name | What It Checks | Phase |
|---|---|---|---|
| 1 | Structural Alignment | Chapter events/tensions vs. chapter plan and locked gates | Phase 2 |
| 2 | Character Consistency | Character behavior vs. committed character frames | Phase 2 |
| 3 | Voice Coherence | Prose voice vs. voice profile + character voice anchors | Phase 1 |
| 4 | Continuity Check | Prose facts vs. continuity ledger; flag candidate new facts | Phase 1 (read-only) / Phase 2 (read-write) |
| 5 | Prose Quality | AI-isms, diction, rhythm, cliches, echoes, readability/pacing | Phase 1 |

**Override rules:**
- Dispatch can reorder passes based on concern tag (e.g., `sounds-too-ai` runs pass 5 first)
- Dispatch can skip passes entirely (e.g., `quick-polish` runs only pass 5 compact)
- Passes 1-2 are automatically skipped when no planning artifacts are available
- If pass 5 finds 10+ hold-severity issues, pause and present findings before running further passes

**Per-pass checkpoint:** After each pass completes, briefly report finding count by severity. If findings are minimal (<3 total), note this and consider skipping lower-priority passes.

### Feature 2 — Concern-Based Dispatch

**Why dispatch matters:** The prose editing toolkit has 9 modules. Running all of them on every text produces a wall of feedback that overwhelms writers. Dispatch maps the writer's stated concern to the minimum effective module set.

**Dispatch logic (in SKILL.md):**

The writer selects a concern tag at intake. The dispatch map determines which passes and which sub-modules within Pass 5 (Prose Quality) activate:

- `sounds-too-ai` — AI-ism Detection (full), Diction Audit (compact), Sentence Variation (compact)
- `needs-tightening` — Diction Audit (full), Cliche Scanner (compact), Echo Detection (compact)
- `rhythm-off` — Sentence Variation (full), Echo Detection (compact), Readability/Pacing (if fiction/long-form)
- `full-review` — Full Prose Lint (all sub-modules, full reference versions)
- `quick-polish` — Full Prose Lint (compact version), capped at top 5 highest-severity findings

**Sub-module loading:** Pass 5's reference file (`prose-lint-modules.md`) contains all six sub-modules consolidated. Dispatch determines which sub-modules activate — unused sub-modules are not processed, preserving context budget.

### Feature 3 — Three Edit Modes (Anti-Ghostwriting Compliant)

**Why three modes:** The original Prose Editing Toolkit offered "rewrite directly." This violates the anti-ghostwriting constraint. The three compliant modes provide escalating levels of guidance while keeping the writer as author.

| Mode | What It Does | When to Use |
|---|---|---|
| **Flag-only** (default) | Identifies issue, names pattern, cites severity. No fix suggestion. | Writer wants to see problems and fix them independently |
| **Flag-with-diagnostic** | Adds WHY explanation referencing craft principles, voice profile, and genre norms | Writer wants to understand the problem before fixing |
| **Flag-with-structural-alternative** | Describes the direction of a fix without providing replacement text | Writer wants direction without dictation |

**Hard boundary:** In ALL three modes, prose-editor never outputs replacement prose. "Change this to [new sentence]" is prohibited. "This sentence hedges — your voice profile shows direct assertions as your baseline" is permitted. "Consider restructuring as a concrete observation rather than a qualification" is permitted.

**Example outputs by mode:**

Flag-only:
> Line 47: AI-ISM — hedging phrase. Severity: Flag.

Flag-with-diagnostic:
> Line 47: AI-ISM — "it seems like perhaps this could indicate" is a hedging chain (4 qualifiers in 7 words). Your voice profile baseline is direct and declarative. This construction departs from that pattern and reads as machine-generated. Severity: Flag.

Flag-with-structural-alternative:
> Line 47: AI-ISM — hedging chain. Your voice is direct. Fix direction: identify the specific claim this sentence is making and state it without qualification. If uncertainty is the point, name what's uncertain and why. Severity: Flag.

### Feature 4 — Voice Profile System

**Why voice profiles:** The single biggest risk in AI-assisted editing is voice drift — the writer's distinctive patterns get smoothed into generic "helpful assistant" prose. Voice profiles create a measurable baseline.

**Two creation paths:**

**Path A — Story-architect stub (at Character Commitment gate):**
- 3-4 friction questions about project-level tone, formality, and rhythm preferences
- Creates a minimal `voice-profile.md` with writer-stated preferences
- Completed during planning before any prose exists
- Useful as a reference but less accurate than Path B (self-description may not match actual practice)

**Path B — Prose-derived (at first prose-editor run):**
- Analyzes submitted text to identify: sentence length tendency, formality register, diction range, rhythm pattern, characteristic constructions
- Presents derived profile to writer for confirmation/modification
- Writer confirms, edits, or rejects each characteristic
- Only confirmed characteristics are written to `voice-profile.md`
- More accurate (based on actual writing) but requires existing prose

**Voice Profile artifact schema:**

```
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

**Voice deviation protocol (Phase 2):**
Writers can mark passages as intentional departures from their profile. prose-editor acknowledges the annotation and skips voice-drift flagging for those passages.

### Feature 5 — Severity Model (Dual-Axis)

**Severity (issue property):**

| Level | Meaning | Review Impact |
|---|---|---|
| **Hold** | Contradicts locked artifact, breaks continuity, or produces reader confusion | Blocks review closure — requires writer acknowledgment |
| **Flag** | Degrades quality but does not create contradictions | Does not block closure |
| **Advisory** | Stylistic observation; writer may legitimately disagree | Does not block closure |

**Intensity (delivery property):**

| Level | Tone | When |
|---|---|---|
| **Gentle** | Reflective, observational | Writer chose gentle; early relationship |
| **Standard** | Direct naming + brief explanation | Default |
| **Firm** | Prioritized, consequence-aware | Writer chose firm; Workshop mode |
| **Diagnostic** | Stop-and-redirect, structural reframing | Premise-level problem detected |

**Cross-axis rules:**
- Holds are never delivered gently (minimum standard intensity)
- Advisories can be delivered at any intensity
- Diagnostics override the selected intensity — if a diagnostic-level problem is found, it's delivered at diagnostic intensity regardless of the writer's setting

| Issue Severity | Gentle | Standard | Firm | Diagnostic |
|---|---|---|---|---|
| Advisory | Noted lightly | Named briefly | Clear recommendation | Only if relevant to diagnosis |
| Flag | Surfaced as observation | Stated with impact | Stated with priority | Included in diagnostic framing |
| Hold | Surfaced clearly (holds are never hidden) | Named with blocking status | Named with consequence chain | Triggers diagnostic intervention |

### Feature 6 — Genre and Audience Calibration

**Genre calibration:**

| Genre | Tolerance | Priority | Special Rules |
|---|---|---|---|
| Fiction | Fragments, unconventional punctuation, stylistic passive OK | Show-don't-tell checks active; pacing analysis active | Don't flag intentional rule-breaking |
| Business | Low tolerance for passive, cliches, wordiness | Clarity and concision priority | Skip show-don't-tell |
| Essay/opinion | Preserve strong voice and personality | Flag hedging aggressively — opinion needs conviction | Longer sentences OK if earned |
| Marketing | Very low tolerance for cliches | Specificity and concrete claims priority | Short paragraphs expected |
| Academic | Passive voice and hedging OK where epistemically appropriate | Clarity and precision over personality | Don't penalize complexity for expert audience |

**Audience calibration (readability targets):**

| Audience | Target Grade Level |
|---|---|
| Adolescent / young adult | 6-8 |
| General public | 10-12 |
| Professional peers | 10-12 |
| Academic reviewers | 12-16 |
| No audience specified | 10-12 (default) |

**Calibration rule:** Don't tell an academic their prose is "too complex" when their audience expects complexity. Genre and audience override raw readability scores.

---

## Section 4 — Memory and Control Structure

### 4.1 — Artifact Model

prose-editor interacts with three categories of artifacts:

**Read-only artifacts** (owned by story-architect, never modified by prose-editor):
- `story-manifest.md` — premise, genre tag, thematic argument
- `structure-map.md` — act structure, turning points, pacing notes
- `characters/[slug].md` — character frames, voice anchors, relationships
- `chapters/[nn]-[slug].md` — chapter plans (scene objective, entry/exit state)
- `handoff-context-v[N].json` — gate statuses, artifact index, open questions

**Read-write artifacts** (owned by prose-editor):
- `voice-profile.md` — project-level voice baseline (created/updated by prose-editor)
- `chapters/[nn]-[slug]-review.md` — per-chapter review findings
- `continuity-ledger.md` — Phase 1: read-only. Phase 2: prose-editor writes confirmed facts

**Transient artifacts** (session-only, not persisted to disk except via review artifact checkpoints):
- Review working state (current pass, accumulated findings, checkpoint decisions)
- Intake context block (genre, audience, concern, edit mode)

**Ownership rule:** prose-editor never writes to story-architect artifacts. If prose-editor detects a structural conflict, it escalates (via writing-guide) rather than modifying the planning artifact. The writer decides whether to revise the plan or the prose.

### 4.2 — Review Artifact Schema

**Structured header fields:**

| Field | Type | Required | Notes |
|---|---|---|---|
| `project` | string | yes | Project slug |
| `chapter` | int | yes | Chapter number |
| `reviewed_at` | ISO date | yes | Review date |
| `entry_mode` | enum | yes | `context-only` or `post-handoff` |
| `concern` | enum | yes | Concern tag from intake |
| `edit_mode` | enum | yes | `flag-only`, `flag-with-diagnostic`, `flag-with-structural-alternative` |
| `intensity` | enum | yes | `gentle`, `standard`, `firm`, `diagnostic` |
| `genre` | string | no | Genre tag or "unspecified" |
| `audience` | string | no | Audience or "unspecified" |
| `voice_profile` | string | no | Path or "none" |
| `passes_run` | int[] | yes | List of pass numbers executed |
| `passes_requested` | int[] | yes | List of pass numbers requested by dispatch |
| `last_completed_pass` | int | yes | Last fully completed pass (for session resume) |
| `handoff_version` | string | no | Handoff file version if post-handoff mode |
| `iteration` | int | yes | Review iteration number (1-3) |

**Freeform body sections:**

```
### Findings

#### Hold (N issues)
[findings with line references, pattern names, severity justification]

#### Flag (N issues)
[findings]

#### Advisory (N issues)
[findings]

### Summary
- Total: N issues (X hold, Y flag, Z advisory)
- Top 3 highest-impact changes: [list]
- Primary diagnosis: [AI patterns | prose weakness | rhythm | diction | structural | voice drift]

### Continuity Candidates
[facts flagged for potential ledger entry — Phase 1: informational only]

### Pass Notes
[per-pass observations, skip declarations, checkpoint decisions, write notices/receipts]
```

### 4.3 — Continuity Ledger Behavior

**Phase 1 (read-only):**
- prose-editor reads the existing continuity ledger (if any) during Pass 4
- Flags prose that contradicts a ledger entry as Hold severity
- Identifies candidate new facts in the prose but does NOT write them to the ledger
- Candidate facts are listed in the review artifact under "Continuity Candidates"
- Writer can manually add candidates to the ledger between sessions
- **Phase 1 guard:** prose-editor must not open continuity-ledger.md with write intent. If the skill detects itself about to write ledger entries, it must halt and redirect candidates to the review artifact's Continuity Candidates section.

**Phase 2 (read-write):**
- Same as Phase 1, plus:
- After review, prose-editor presents candidate facts to the writer for confirmation
- Only writer-confirmed facts are written to the ledger with Write Notice/Receipt
- Each ledger entry tracks: fact_id, fact statement, established_in (chapter ref), confirmed_by (writer), status (active/superseded)

### 4.4 — Context Budget Management

**Tiered loading strategy:**

| Tier | What Loads | When | Est. Tokens |
|---|---|---|---|
| Always | SKILL.md, intake context | Session start | ~2500 |
| On review start | `prose-lint-modules.md` (active sub-modules only) | Pass 5 begins | ~800-1500 |
| On voice check | `voice-profile-protocol.md`, `voice-profile.md` | Pass 3 begins | ~700 |
| On structural check | Chapter plan, relevant character files, structure-map | Pass 1-2 begin (Phase 2) | ~1000-2000 |
| On continuity check | `continuity-ledger.md` | Pass 4 begins | ~200-800 |
| On escalation | `dispatch-and-escalation.md` (escalation section) | Escalation triggered | ~400 |
| On post-edit | `review-protocol.md` (verification section) | Verification pass | ~500 |
| Never inline | Full handoff JSON, all character files simultaneously | Load by reference only | — |

**Sub-module selection mechanism:** Load only sub-modules from `prose-lint-modules.md` matching the dispatch concern tag. A `sounds-too-ai` review loads AI-ism detection + diction audit sub-modules (~100 lines), not all six.

**Budget rule:** Same 60% capacity threshold as story-architect. If approaching budget, unload completed pass references before loading next pass. Warn writer if budget prevents running all requested passes — offer to prioritize.

### 4.5 — Anti-Ghostwriting Enforcement

**Behavioral rule (prose-editor specific):**
prose-editor may analyze, flag, categorize, and describe patterns in writer-submitted prose. It may suggest structural fix directions. It may NOT:
- Output replacement sentences, phrases, or words
- Complete unfinished sentences
- Rewrite passages (even if asked)
- Generate example "improved" versions of flagged text
- Produce before/after comparisons with AI-generated "after" text

**Diagnostic creep anti-pattern:** Diagnostics must describe the *category* and *effect* of the problem, not the *specific textual correction*. If a diagnostic contains enough information to mechanically produce the fix without creative judgment, it has crossed from diagnosis into ghostwriting.

**Redirect pattern for indirect requests:**
When a writer says "just fix it" or "rewrite this for me," prose-editor responds:
> "I can flag what's not working and explain why, but the words are yours. Want me to run a [concern-appropriate] review so you can see exactly what to target?"

**What IS permitted:**
- Naming the pattern: "This is a hedging chain"
- Explaining the principle: "Direct assertions read stronger than qualified ones"
- Describing the direction: "This passage would benefit from specificity over qualification"
- Referencing the voice profile: "Your baseline is [X]; this passage departs from it"
- Single-word category labels: "NOMINALIZATION", "AI-ISM", "ECHO"

**Local enforcement:** prose-editor enforces `ghostwriting-risk` locally, consistent with the system-wide anti-ghostwriting pattern documented in escalation-triage.md. No escalation pathway — the skill refuses the action directly.

### 4.6 — Handoff Loading (Post-Handoff Entry)

**Schema authority:** The handoff-context.json schema is defined authoritatively in story-architect's `references/artifact-schemas.md`. prose-editor consumes this contract; it does not define it.

**When entering with handoff:**
1. Locate latest `handoff-context-v[N].json` in `docs/writing/[project]/handoff/`
2. Validate `schema_version` MAJOR on load — MAJOR mismatch triggers error surfaced to writer, not silent degradation. Offer context-only mode as fallback.
3. Extract: `project.genre_tag`, `gates` (status of each), `artifacts` (paths), `session_mode`, `open_questions`
4. Load locked gate fields as hard constraints (do not flag prose that complies with locked decisions)
5. Load artifact paths — read on demand per pass, not all at once
6. Surface `open_questions` with `priority: high` to the writer before starting review

**Edge cases:**
- All gates `not_started` or `skipped`: proceed with advisory note that planning foundation is thin. Treat all findings as advisory or flag, never hold (no locked constraints to violate).
- No text provided: soft rejection — prompt writer to supply text before review can begin.

**Version mismatch detection (Phase 2):**
Compare `handoff_id` field between loaded version and latest file. If ID changed, alert writer. Writer chooses: review against new version, or continue with original. Decision recorded in review artifact header.

### 4.7 — Artifact Write Notice/Receipt Protocol

prose-editor follows the same Artifact Write Notice / Write Receipt protocol as story-architect:
- **Write Notice** (before write): names the file and what changed
- **Write Receipt** (after write): confirms the file path
- Both logged in the review artifact's Pass Notes section

This applies to all prose-editor artifact writes: voice-profile.md, review artifacts, and continuity ledger entries (Phase 2).

### 4.8 — Review Checkpoint/Recovery

After each pass completes, prose-editor writes accumulated findings to the review artifact on disk. This serves as a checkpoint. If session resumes:
1. Load partial review artifact
2. Read `last_completed_pass` field
3. Continue from the next incomplete pass
4. Present resume summary to writer: "Review in progress — [N] of [M] passes complete, [X] findings so far. Continue from Pass [N+1]?"

### 4.9 — Escalation Payload Schema

**Direction 3: prose-editor escalates to writing-guide**

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

This payload schema must be added to `artifact-schemas.md` as "Direction 3: prose-editor escalates to writing-guide."

**Concurrent escalation rule:** If multiple escalations are triggered in the same review pass, holds take priority. Hold-severity escalations are emitted as payloads; advisory-severity findings are bundled into the review artifact but not emitted as escalation payloads.

### 4.10 — Anti-Drift Controls

**Cross-pass consistency check:** Before writing findings from Pass N, compare against findings from Passes 1..N-1 for contradictions. If Pass 3 (voice) flags a passage as "too formal" but Pass 5 (prose quality) flags the same passage as "appropriately precise for the genre," surface the contradiction as a note in Pass Notes rather than silently presenting contradictory findings.

**Writer-disputed findings:** When a writer challenges a finding during Issue Triage, the finding is marked "disputed" in the review artifact — not removed. Disputed findings are excluded from the summary's issue count but remain visible for reference. This preserves the diagnostic record while respecting the writer's judgment.

---

## Section 5 — Integration Contracts

### 5.1 — prose-editor Entry Contract

All entry paths go through writing-guide. prose-editor is never invoked directly.

| Entry Path | Trigger | What Writing-Guide Does | What prose-editor Receives |
|---|---|---|---|
| **Explicit invocation** | Writer pastes prose and requests review | Offers prose-editor; writer confirms | Submitted text + context-only mode |
| **Post-handoff** | story-architect completes handoff | Surfaces handoff availability; writer confirms | Submitted text + handoff-context.json |
| **Diagnostic escalation** | writing-guide detects prose-level problem beyond craft guidance scope | Offers prose-editor with escalation payload | Escalation payload + writer's text |
| **Session resume** | Incomplete review artifact detected | Offers to continue from last completed pass | Partial review artifact + writer's text |

**Context-only vs. post-handoff capability:**

| Capability | Context-Only | Post-Handoff |
|---|---|---|
| Passes 1-2 (structural, character) | Skipped with declaration | Available (Phase 2) |
| Pass 3 (voice coherence) | Runs if voice profile exists | Runs against voice profile + character voice anchors |
| Pass 4 (continuity) | Skipped unless ledger exists from prior review | Runs against existing ledger |
| Pass 5 (prose quality) | Full capability | Full capability + genre calibration from handoff |
| Review artifact header | `entry_mode: context-only` | `entry_mode: post-handoff` |

**Quality threshold for context-only mode:** When passes are skipped, the review artifact declares them explicitly: "Passes 1, 2 skipped — no planning artifacts available. This review covers prose quality and voice consistency only."

### 5.2 — Handoff Contract (story-architect to prose-editor)

**What prose-editor reads from handoff-context.json:**

| Field | How prose-editor Uses It |
|---|---|
| `project.genre_tag` | Selects genre calibration for all passes |
| `gates.premise.status` | If locked, premise compliance is a hard constraint |
| `gates.characters.status` | If locked, character consistency checks reference committed frames |
| `gates.voice_profile.status` | If locked, voice profile is the review baseline |
| `artifacts.character_profiles[]` | Paths to load for Pass 2 (character consistency) |
| `artifacts.voice_doc` | Path to voice-profile.md for Pass 3 |
| `open_questions[]` | Surfaced to writer before review starts (high-priority only) |
| `session_mode.gate_surfacing` | Controls when conflicts with locked gates are raised |

**What prose-editor does NOT read:** `handoff_notes` (skill-generated metadata, not writer content).

### 5.3 — Escalation Contract (prose-editor to writing-guide)

| Trigger | Condition | Severity | Routes To |
|---|---|---|---|
| `voice-drift` | Sustained voice departure across 3+ passages | Advisory | writing-guide (present KB craft guidance) |
| `structural-conflict` | Prose contradicts locked gate field | Hold | writing-guide, then writer decision (plan-revise or prose-revise) |
| `continuity-contradiction` | Prose contradicts established ledger fact | Hold | writing-guide, then writer decision |
| `diagnostic` | Premise-level problem detected in prose | Hold | writing-guide (Diagnostic mode activation) |

**prose-editor does NOT:**
- Modify planning artifacts
- Route directly to story-architect (always goes through writing-guide)
- Resolve conflicts — presents them for writer decision

**Type reconciliation with escalation-triage.md:** `structural-conflict` is distinct from story-architect's `structural-gap`. A gap is a missing required field; a conflict is prose contradicting a locked field. Both route through writing-guide but with different triage logic. `structural-conflict` must be added to `escalation-triage.md` with its own routing rules.

### 5.4 — Return Loop Contract

When a writer revises planning artifacts after a prose-editor escalation:

1. Writer re-enters story-architect, revises at relevant gate
2. story-architect generates `handoff-context-v[N+1].json`
3. On next prose-editor invocation, version mismatch detected (compare `handoff_id`)
4. prose-editor asks writer: "Planning artifacts updated. Review against new version or continue with previous baseline?"
5. If new version: scoped re-review of previously-flagged structural issues only
6. If previous: continue with original review context (recorded in review artifact)

**Writer-decline path:** If the writer declines re-review, the review artifact records which handoff version it was reviewed against.

**Loop-break:** Maximum 2 re-review cycles per review session. After 2, writer must explicitly restart the full review to continue.

### 5.5 — Writing-guide Routing Update

When prose-editor ships, writing-guide requires these minimal updates:

**specialist-routing-guide.md changes:**

1. **Section 1 (Decision Tree):** Remove `[FUTURE -- not yet available]` from prose-editor branch. Priority: if both story-architect and prose-editor could match, story-architect takes priority (planning before review).

2. **Section 3 (Entry Triggers):** Replace FUTURE placeholder with:
   - Explicit invocation — Writer pastes prose and requests review
   - Post-handoff invocation — story-architect handoff exists
   - Diagnostic escalation — Structural module surfaces prose-level problem
   - Session resume — Incomplete review artifact detected

3. **Section 5 (Negative Heuristics):** Add: "prose-editor does not handle general craft questions about technique, genre selection, or story structure — those stay in writing-guide."

4. **Section 6 (Session State Detection):** Add: "Check for incomplete `[nn]-[slug]-review.md` in `docs/writing/[project]/chapters/` directory at session start."

**Detection heuristics (primary signal: keyword match, secondary: text length):**
- Writer mentions: "review," "edit," "lint," "sounds like AI," "voice," "rhythm," "too wordy"
- Writer pastes a block of prose (>50 words) alongside a review request
- Writer has an active story-architect project with completed handoff and asks about their draft

**Additional file updates:**
- `escalation-triage.md` — Add `structural-conflict` type with routing rules. Add Direction 3 payload schema reference.
- `ROUTER.REVISION.md` (KB) — Add two routing entries for new leaf guides (`GUIDE.PROSE.AI_PATTERN_DETECTION.md`, `GUIDE.PROSE.VOICE_CONSISTENCY.md`).

---

## Section 6 — Initial Implementation Model

### 6.1 — Skill File Structure

**Three-level progressive disclosure:**

```
~/.claude/skills/prose-editor/
  SKILL.md                              # ~430 lines (under 500 limit)
  references/
    prose-lint-modules.md               # ~250 lines — Modules 1-6 consolidated, adapted
    voice-profile-protocol.md           # ~120 lines — Module 7 adapted + deviation protocol
    review-protocol.md                  # ~280 lines — five-pass logic + post-edit verification
    dispatch-and-escalation.md          # ~170 lines — dispatch map + escalation payloads
```

**Why four reference files (not six):** Consolidation follows the pattern established by story-architect, where every reference file exceeds 100 lines and carries conceptual weight. Three originally-proposed files under 100 lines were merged: post-edit-verification into review-protocol (verification is the last step of the review flow), escalation-payloads and module-dispatch-map into dispatch-and-escalation (dispatch and escalation are both routing concerns).

**SKILL.md contains:**
- Hard boundaries (5 rules, non-bypassable)
- Entry points (4 paths via writing-guide)
- Checkpoint sequence (intake, review, triage, resolution)
- Dispatch logic (concern tag to pass selection)
- Edit mode definitions (3 modes with behavioral rules)
- Severity model (dual-axis tables)
- Anti-ghostwriting enforcement + diagnostic creep anti-pattern + redirect patterns
- Reference file load triggers (one-line conditions per file)

**Reference file load conditions:**

| File | Load Trigger | Unload Trigger |
|---|---|---|
| `prose-lint-modules.md` | Pass 5 begins (active sub-modules per dispatch) | Pass 5 complete |
| `voice-profile-protocol.md` | Pass 3 begins OR Voice Profile creation | Pass 3 complete / creation done |
| `review-protocol.md` | Review checkpoint begins | Writer Resolution checkpoint clears |
| `dispatch-and-escalation.md` | Intake (dispatch section) OR escalation triggered (escalation section) | After dispatch decision / after payload emitted |

### 6.2 — KB Extensions

Two new leaf guides added to writing-guide's KB:

```
docs/superpowers/writing-kb/writing_mentor_kb/02_leaf_guides/prose/
  GUIDE.PROSE.AI_PATTERN_DETECTION.md   # ~150 lines
  GUIDE.PROSE.VOICE_CONSISTENCY.md      # ~100 lines
```

**GUIDE.PROSE.AI_PATTERN_DETECTION.md** — Distilled from Toolkit Module 1. Covers: AI cliche phrases (catalog), formulaic structure patterns, hedging/weasel patterns, tonal flatness indicators, transition overuse thresholds, punctuation tells. Routable from `ROUTER.REVISION.md` when symptom is "sounds AI" or "generic voice."

**GUIDE.PROSE.VOICE_CONSISTENCY.md** — Craft guide for understanding voice drift. Covers: what voice is (vs. style, tone, register), common drift patterns, how AI editing causes homogenization, the role of intentional variation. Routable from `ROUTER.REVISION.md` when symptom is "lost my voice" or "sounds flat."

**KB router update:** `ROUTER.REVISION.md` needs two new routing entries pointing to these guides. Both are Phase 1 deliverables.

### 6.3 — Checkpoint Structure

**Why checkpoints, not gates:** prose-editor's workflow is batch review, not turn-by-turn gap resolution. Gates (as used by story-architect) require field-level tracking, friction scans, and required-field resolution. Checkpoints are ordered phases — lighter, appropriate for diagnostic review where the writer's text is the input, not the writer's answers to structured questions.

**Four checkpoints:**

| Checkpoint | Trigger | Writer Sees | Writer Provides | Artifact Written | On Rejection/Failure |
|---|---|---|---|---|---|
| **Intake Check** | Session start | Concern tag options, edit mode options, genre/audience prompts (or handoff summary) | Concern tag, edit mode selection (or confirmation of handoff context) | Review artifact header created | Cannot proceed without concern tag — re-prompt |
| **Review Pass** | Each pass execution | "Pass [N] complete: X hold, Y flag, Z advisory" | Nothing required (informational) | Findings appended to review artifact on disk | If 10+ holds: pause, present findings, ask whether to continue |
| **Issue Triage** | All passes complete | Consolidated findings by severity, holds listed first | Acknowledge each hold (accept, defer, or escalate) | Holds marked acknowledged/deferred/escalated in artifact | Unacknowledged holds block review closure — re-prompt |
| **Writer Resolution** | Triage complete | Options: done, another pass, target specific, escalate | Selection | Review artifact closed (if done) or iteration incremented | Over-editing warning on 3rd iteration |

**Checkpoint ordering:** Strictly linear — Intake fires once, Review Pass repeats per pass, Issue Triage fires once after all passes, Writer Resolution fires once. On iteration (writer submits revised text), the cycle restarts from Review Pass with a new iteration number.

### 6.4 — Session Resume Decision Tree

1. Check for incomplete review artifact in `docs/writing/[project]/chapters/`
2. If found: read `last_completed_pass` and `passes_requested`
3. Present resume summary: "Review in progress for Chapter [N] — [X] of [Y] passes complete, [Z] findings so far."
4. Writer chooses:
   - **Resume** — continue from next incomplete pass with existing context
   - **Restart** — discard partial artifact, begin fresh review
   - **Review findings so far** — show current artifact contents, then re-ask

On resume: load partial review artifact, reload relevant reference files for the next pass, continue.

### 6.5 — Phase 1 vs. Phase 2 Capability Map

| Capability | Phase 1 | Phase 2 |
|---|---|---|
| Pass 1 (Structural Alignment) | Skipped with declaration | Active |
| Pass 2 (Character Consistency) | Skipped with declaration | Active |
| Pass 3 (Voice Coherence) | Active | Active + voice deviation annotations |
| Pass 4 (Continuity) | Read-only | Read-write |
| Pass 5 (Prose Quality) | Full capability | Full capability |
| Voice Profile (Path A: stub) | Available | Available |
| Voice Profile (Path B: prose-derived) | Available | Available |
| Handoff loading | Reads artifacts as context | Full structural checking + version mismatch |
| Escalation: voice-drift | Available | Available |
| Escalation: diagnostic | Available | Available |
| Escalation: structural-conflict | Not available (no structural checks) | Available |
| Escalation: continuity-contradiction | Available (read-only ledger can detect contradictions) | Available |
| Session resume | Basic (resume from last pass) | + stale artifact tiers |
| KB leaf guides | Both (Phase 1 deliverable) | Same |
| Routing update | Phase 1 deliverable | Same |
| escalation-triage.md update | Partial (voice-drift, diagnostic types) | Full (+ structural-conflict type) |
| Anti-drift controls | Cross-pass consistency, disputed findings | Same |
| Write Notice/Receipt | Active | Active |

### 6.6 — Artifact Taxonomy

| Artifact | Category | Written By | Read By |
|---|---|---|---|
| `voice-profile.md` | Project (persistent) | prose-editor (writer-confirmed) | prose-editor, writing-guide |
| `[nn]-[slug]-review.md` | Review (persistent) | prose-editor | writing-guide (triage), writer |
| `continuity-ledger.md` | Project (persistent) | Phase 1: story-architect only. Phase 2: + prose-editor (writer-confirmed) | prose-editor, story-architect |
| Escalation payloads | Transient (conversation) | prose-editor | writing-guide (triage) |
| Intake context | Transient (session) | prose-editor | prose-editor |

### 6.7 — Writing-guide Extensions Required

When prose-editor ships, writing-guide needs these minimal updates:

| File | Change | Phase |
|---|---|---|
| `specialist-routing-guide.md` | Remove `[FUTURE]`, add entry triggers, detection heuristics, negative heuristics, session detection | Phase 1 |
| `escalation-triage.md` | Add `structural-conflict` type, add Direction 3 payload schema reference | Phase 2 (structural-conflict); Phase 1 (Direction 3 reference) |
| `ROUTER.REVISION.md` (KB) | Add routing entries for two new leaf guides | Phase 1 |
| `artifact-schemas.md` | Add Direction 3 escalation payload schema | Phase 1 |

No changes to writing-guide's SKILL.md, intensity modes, or other reference files.

### 6.8 — Implementation Prerequisites

Before prose-editor can be built, these must exist:

1. **Lint module definitions** — Toolkit Modules 1-6 consolidated and adapted into `prose-lint-modules.md`
2. **Voice profile protocol** — Module 7 adapted with both creation paths
3. **Review protocol** — Five-pass logic with post-edit verification
4. **Dispatch and escalation** — Concern-to-pass mapping and Direction 3 payload schemas
5. **Direction 3 payload schema in artifact-schemas.md** — Must be added before escalation can be tested
6. **Two KB leaf guides** — GUIDE.PROSE.AI_PATTERN_DETECTION.md and GUIDE.PROSE.VOICE_CONSISTENCY.md
7. **ROUTER.REVISION.md update** — New routing entries for the leaf guides

---

## Section 7 — Risks, Tensions, and Open Design Questions

### 7.1 — Overview

This section documents the known trade-offs in prose-editor's design, implementation risks that could surface during build, questions resolved during this spec session, and questions deferred to future work. The tensions are genuine design trade-offs where both sides have merit — not problems to be solved, but boundaries to be managed.

### 7.2 — Design Tensions

**Tension 1: Diagnostic depth vs. ghostwriting boundary**

prose-editor must provide enough detail for the writer to understand AND fix the problem, but not so much that the fix is effectively dictated. The three edit modes escalate from "here is the problem" (flag-only) through "here is why it's a problem" (flag-with-diagnostic) to "here is what kind of fix would address it" (flag-with-structural-alternative). Each escalation step brings prose-editor closer to the ghostwriting boundary.

The diagnostic creep anti-pattern (Section 4.5) defines the line: if a diagnostic contains enough information to mechanically produce the fix without creative judgment, it has crossed into ghostwriting. But "enough information to mechanically produce" is itself a judgment call. A description like "this passage would benefit from specificity" is safely abstract; "the subject should perform the action" is dangerously close to a syntactic instruction.

Residual risk: even with the anti-pattern defined, model behavior may drift toward specificity over time. This tension requires ongoing vigilance, not a one-time resolution.

**Tension 2: Review thoroughness vs. feedback overwhelm**

Running all five passes on a chapter can produce 40+ findings spanning AI-isms, diction weaknesses, rhythm monotony, echoes, pacing problems, and structural conflicts. Presenting all findings at once is paralyzing. But filtering too aggressively risks missing important patterns that only become visible across multiple categories.

The dispatch system mitigates this by mapping the writer's stated concern to a targeted pass set. The per-pass checkpoint mitigates it further by pausing on 10+ holds. The quick-polish mode caps at 5 findings. But a `full-review` with `flag-with-diagnostic` mode on a rough draft will still produce dense output.

Residual risk: writers may avoid `full-review` because output is overwhelming, defaulting to `quick-polish` and missing significant issues. The per-pass checkpoint is the primary safety valve.

**Tension 3: Voice profile accuracy vs. voice evolution**

A voice profile captures the writer's patterns at a point in time. Writers evolve — and should evolve. A profile derived from Chapter 1 may not represent the writer's voice by Chapter 20. Enforcing strict voice consistency could suppress natural growth or penalize deliberate stylistic development.

Phase 1 mitigates this by treating voice drift findings as Flag severity (not Hold) — the writer sees the observation but is not blocked. Phase 2 adds voice deviation annotations, allowing writers to mark passages as intentional departures. But between Phase 1 and Phase 2, there is no mechanism for the writer to update their voice profile mid-project except by re-running Path B derivation.

Related: OQ2 (original spec) — intentional voice deviation protocol. This spec's Phase 2 resolution addresses prose-editor's scope; the parent spec's OQ2 remains open for the broader ecosystem.

**Tension 4: Genre calibration precision vs. genre-agnostic simplicity**

The system is designed to be genre-agnostic at the structural level (same gate sequence, same artifact schemas, genre vocabulary substitution). But prose quality review IS genre-dependent — passive voice is fine in academic writing, problematic in marketing copy. Over-calibrating the genre system risks complexity; under-calibrating risks false positives that erode writer trust.

The genre calibration table (Section 3, Feature 6) provides coarse-grained calibration. But edge cases exist: a literary novel with sections written as fake academic papers, a business report with intentionally narrative storytelling, a memoir that shifts register between chapters.

Residual risk: genre calibration is necessarily approximate. The "don't flag legitimate style" rule is the escape valve, but its application requires judgment that may vary across sessions.

**Tension 5: Multi-POV voice consistency**

When a novel has multiple POV characters, each with their own voice anchors, prose-editor must apply per-character voice checking to the relevant sections. This creates a combinatorial review surface: the project voice profile applies everywhere, but character voice anchors apply only in their sections. A passage that violates the project voice may be perfectly consistent with a character's established voice.

The resolution (OQ-PE4) is writer-annotated POV sections. But annotation requires writer effort, and unannotated multi-POV chapters fall back to project-level review only, potentially missing character-voice inconsistencies or flagging intentional character-voice departures as drift.

**Tension 6: Inter-skill diagnostic boundary**

When prose-editor's findings strongly imply a planning artifact is wrong (e.g., a character's behavior in prose consistently contradicts their committed arc), the skill must restrain itself to flagging rather than diagnosing the planning problem. prose-editor does not write to planning artifacts and does not route directly to story-architect. It escalates through writing-guide.

This indirection is by design — the writer must decide whether to revise the plan or the prose. But it means prose-editor cannot offer the most helpful diagnosis ("your character arc needs revision") because that crosses into story-architect's domain. The escalation payload can only say "conflict detected between prose and committed arc."

### 7.3 — Implementation Risks

**Risk 1: prose-lint-modules.md context consumption**

At ~250 lines (~1500 tokens for the full file), loading the entire consolidated module file for every Pass 5 invocation consumes significant context. When the writer submits a long chapter (5000+ words), context pressure compounds. If the writer also has planning artifacts loaded from a handoff, the combined context may approach the 60% threshold before the review even begins.

Mitigation: sub-module selection loads only the sub-modules matching the dispatch concern tag. A `sounds-too-ai` review loads ~100 lines, not ~250. But `full-review` loads all sub-modules, and there is no way to avoid this for comprehensive reviews.

**Risk 2: Review artifact sprawl**

Detailed findings with line references, pattern names, severity justifications, and pass notes produce large review artifacts. A thorough `full-review` with `flag-with-diagnostic` mode could produce a 200+ line review artifact for a single chapter. Over multiple iterations (up to 3), the per-chapter review output can grow large.

Mitigation: each iteration is a new review artifact (not appended to the old one). Previous iteration artifacts are retained for comparison but not loaded into active context.

**Risk 3: Escalation payload interpretation drift**

prose-editor's escalation payloads must be interpretable by writing-guide's triage layer. If the payload schema evolves independently of the triage rules (e.g., prose-editor adds a new `conflict_type` value that triage does not handle), routing breaks silently.

Mitigation: `artifact-schemas.md` as single source of truth for all payload schemas. Schema version field for forward-compatibility. But schema governance is only as strong as the discipline to update it — this is an operational risk.

**Risk 4: Voice Profile over-reliance**

If writers treat the voice profile as a style guide rather than a diagnostic baseline, they may resist legitimate voice evolution or become dependent on prose-editor's validation. The profile should inform, not constrain.

Mitigation: voice profile explicitly labeled "diagnostic baseline, not a prescription." The Preserve List captures what to protect; the Watch List captures what to improve. Neither is a rule. But the framing may not prevent writers from treating it as one.

**Risk 5: Context window pressure during full review**

A full five-pass review with planning artifacts, voice profile, lint modules, and continuity ledger loaded simultaneously could exhaust the context window. The tiered loading strategy (Section 4.4) mitigates this by loading and unloading per pass, but Passes 1-2 (Phase 2) require planning artifacts that overlap with Pass 3's voice profile and Pass 4's continuity ledger.

Mitigation: 60% capacity threshold with writer notification. Unload completed pass references before loading next pass. Offer to prioritize passes if budget is insufficient.

**Risk 6: Context-only mode quality threshold**

Context-only mode skips Passes 1-2 and may skip Pass 4 (if no ledger exists). This leaves only Passes 3 and 5 — voice coherence and prose quality. For a writer who has not used story-architect, this is the only available mode. The question is whether a voice + prose quality review alone is genuinely useful, or whether it produces review shallow enough to mislead the writer into thinking their text has been comprehensively checked.

Mitigation: review artifact declares skipped passes explicitly. But the mitigation relies on the writer reading and understanding the declaration, which is not guaranteed.

Related: OQ-PE10 (remaining question) — should context-only mode carry a quality disclaimer?

**Risk 7: Recurring-finding fatigue**

If a writer makes the same class of error across multiple chapters (e.g., consistent AI-ism patterns, habitual hedging), prose-editor will flag the same patterns repeatedly. After the third chapter review with identical findings, the writer may stop reading the feedback.

Mitigation: not yet specified. A future enhancement could track cross-chapter patterns and escalate to writing-guide for craft education rather than repeated flagging.

**Risk 8: Staged build expectations**

Phase 1 skips structural alignment and character consistency — the passes with highest value for writers who used story-architect. Writers who spent hours planning may expect prose-editor to immediately check against those plans.

Mitigation: explicit skip declarations in review artifacts. Phase 1 still loads planning artifacts as context for prose quality passes (informed review even without dedicated structural checks). Clear documentation that Phase 2 adds these capabilities.

### 7.4 — Resolved Open Questions

| OQ | Question | Resolution | Scope |
|---|---|---|---|
| OQ-PE1 | How does the read-only continuity ledger present flagged facts? | Candidate facts listed in review artifact under "Continuity Candidates" section — informational only in Phase 1, writer-confirmable in Phase 2. | prose-editor |
| OQ-PE2 | Should Voice Profile stub include voice anchors from character files? | No. Voice anchors are per-character; voice profile is project-level. Voice anchors loaded separately during Pass 3 for character-specific checking. | prose-editor |
| OQ-PE3 | Minimum text length for meaningful analysis? | No hard minimum. Structural passes (1-2) need at minimum one complete scene. Prose quality passes work on single paragraphs. prose-editor runs on any length. | prose-editor |
| OQ-PE4 | How to handle multi-POV chapters? | Writer annotates POV sections. prose-editor applies per-character voice anchors to annotated sections. Unannotated text reviewed against project voice profile only. | prose-editor |
| OQ2 | Intentional voice deviation protocol | Phase 2: writers mark passages as intentional departures; prose-editor skips voice-drift flagging for those passages. Resolved for prose-editor's scope. Parent spec OQ2 addressed by this resolution but not formally closed in parent document. | Inherited from story-architect spec |
| OQ6 | Context-only mode quality threshold | Review artifact declares skipped passes explicitly: "Passes 1, 2 skipped — no planning artifacts. This review covers prose quality and voice consistency only." Quality remains a residual risk (see Risk 6). | Inherited from story-architect spec |
| OQ9 | Editing support boundary specification | Three compliant edit modes (flag-only / flag-with-diagnostic / flag-with-structural-alternative). Diagnostic creep anti-pattern defined. Redirect pattern for "just fix it" requests. Resolved for prose-editor's scope. | Inherited from story-architect spec |

### 7.5 — Remaining Open Questions

| OQ | Question | Deferred To | Related |
|---|---|---|---|
| OQ-PE5 | Should prose-editor support batch review (multiple chapters at once)? | Future enhancement after Phase 2 | Risk 7 (recurring-finding fatigue) |
| OQ-PE6 | How should prose-editor handle non-English text? | Future enhancement — current toolkit is English-focused | — |
| OQ-PE7 | Should the continuity ledger be shared across multiple projects in a series? | Future spec for series/multi-book support | — |
| OQ-PE8 | How should voice profile handle collaborative writing (multiple authors)? | Future enhancement | Tension 5 (multi-POV) |
| OQ-PE9 | Session recovery under context pressure — what happens when a mid-review session drops and the context window is too small to resume with all artifacts loaded? | Future enhancement — equivalent to story-architect OQ7 | Risk 5 (context pressure) |
| OQ-PE10 | Should context-only mode carry a quality disclaimer beyond the skip declaration? | Validate after Phase 1 user testing | Risk 6 (context-only quality) |

### 7.6 — Out of Scope

| Area | Rationale |
|---|---|
| Batch/multi-chapter review | Requires cross-chapter pattern tracking not designed in this spec |
| Series-level continuity | Requires multi-project ledger architecture |
| Multi-author voice profiles | Requires per-author profile management and section attribution |
| Non-English language support | Toolkit detection patterns are English-specific; adaptation is non-trivial |
| Real-time/streaming review | Review is diagnostic, not drafting-time — operates on submitted text |
| External tool integration | prose-editor replaces external tools within Claude; integration adds complexity without clear value |
| prose-editor full design within story-architect spec | This standalone spec addresses the gap noted in the story-architect spec's Section 7.5 |
