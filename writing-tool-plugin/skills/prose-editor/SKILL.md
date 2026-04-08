---
name: prose-editor
description: >
  Editorial review and continuity layer for writers actively drafting or
  revising. Use this skill whenever a writer wants to review prose for AI
  patterns, voice consistency, rhythm, diction, or general quality — including
  when they say things like "sounds like AI", "flat prose", "lost my voice",
  "too wordy", or "needs tightening". Also use when a writer returns with
  revised text for post-edit verification, when writing-guide detects a
  prose-level problem exceeding craft guidance scope, or when story-architect
  generates a handoff-context.md for prose review. Analyzes submitted text
  through structured diagnostic passes -- never writes prose or generates
  content.
---

# prose-editor

An editorial review skill that helps writers see what their eye skips — AI-isms, rhythm monotony, diction weakness, echo patterns, pacing problems — without imposing a "correct" style. A diagnostic instrument, not a rewriter.

---

## 1. Hard Boundaries

`Read ../../shared/anti-ghostwriting.md`

Additional prose-editor boundary: prose-editor is a diagnostic lens — it marks where the seams show, names why, and points toward the fix. It never holds the red pen. See Section 9 (Anti-Ghostwriting Enforcement) for detection and response logic.

---

## 2. Entry Points

Two entry paths: direct invocation (`/prose-editor`) or via writing-guide routing.

### Direct invocation
Writer invokes `/prose-editor` directly with prose and a review request. prose-editor handles its own intake.

### Via writing-guide
writing-guide detects a prose-review need and offers prose-editor. This is the discovery path for ambiguous queries.

### Path 1: Explicit invocation
Writer pastes prose and requests review. writing-guide detects prose-review need and offers prose-editor. prose-editor receives: submitted text + context-only mode.

### Path 2: Post-handoff invocation
story-architect generates `handoff-context.md`. Writer enters prose-editor with full planning context. prose-editor receives: submitted text + handoff-context.md.

### Path 3: Diagnostic escalation
writing-guide or story-architect surfaces a prose-level problem (voice drift, rhythm issue) that exceeds craft guidance scope. prose-editor receives: escalation payload + writer's text.

### Path 4: Session resume
writing-guide detects incomplete review artifact (`[nn]-[slug]-review.md` with `last_completed_pass` < total passes) and offers to continue from last completed pass. prose-editor receives: partial review artifact + writer's text.

prose-editor runs 3 review passes: Voice Coherence, Continuity Check, Prose Quality.

### Context-only vs. post-handoff capability

| Capability | Context-Only | Post-Handoff |
|---|---|---|
| Pass 1 (voice coherence) | Runs if voice profile exists | Runs against voice profile + character voice anchors |
| Pass 2 (continuity) | Skipped unless ledger exists | Runs against existing ledger |
| Pass 3 (prose quality) | Full capability | Full capability + genre calibration from handoff |

---

## 3. Checkpoint Sequence

Four checkpoints control prose-editor's flow. They are strictly linear — Review Pass repeats per pass.

### Checkpoint 1: Intake Check
- **Trigger:** Session start
- **Required input:** Concern tag selection, edit mode selection
- **Artifact written:** Review artifact header (intake context block)
- **Failure path:** No text provided → soft rejection, prompt writer to supply text

#### Intake Inference

Infer defaults from context before asking:

| Signal | Inferred Setting |
|---|---|
| Writer says "sounds like AI" | concern: sounds-too-ai, edit: flag-with-diagnostic |
| Writer says "needs tightening" | concern: needs-tightening, edit: flag-only |
| Writer says "check the rhythm" | concern: rhythm-off |
| Writer pastes text with no instruction | concern: full-review |
| handoff-context.md exists | genre, audience from handoff |
| voice-profile.md exists | voice profile path set |

Confirm inferred settings: "Running a [concern] review at [intensity]. Sound right?"
Only prompt for fields that cannot be inferred.

**Concern tags:** `sounds-too-ai` / `needs-tightening` / `rhythm-off` / `full-review` / `quick-polish`

**Edit modes:** `flag-only` (default) / `flag-with-diagnostic` / `flag-with-structural-alternative`

**Intensity:** `gentle` / `standard` (default) / `firm` / `diagnostic` — inherited from session or selected at intake

#### Voice Profile Optionality

Only offer voice profile creation when:
- Concern tag is `full-review`
- Writer explicitly asks about voice
- Session is post-handoff

For `sounds-too-ai`, `needs-tightening`, and `quick-polish`, run without a voice profile. Pass 1 (Voice Coherence) skips with a note: "Voice coherence pass skipped -- no voice profile available. Run with `full-review` to create one."

### First Impression

Before loading any sub-module, run these 5 diagnostic questions against
the submitted text:

1. **Is anyone home?** — Can you detect a specific person making specific choices, or could this have been written by anyone about anything?
2. **Can you see anything?** — Is the language anchored in visible, physical reality, or does it float in abstraction?
3. **Does it vary?** — Do sentence lengths, structures, and rhythms change, or does the prose hum at one frequency?
4. **Does it take a position?** — Does the writer commit to a stance, or present permanent hedged neutrality?
5. **Does anything surprise?** — Is there a single phrase, image, or structural choice you did not see coming?

Share the assessment with the writer before granular findings.

If all 5 negative: recommend escalating to full-review regardless of concern tag.
If all 5 positive: recommend quick-polish (text may only need minor attention).

### Checkpoint 2: Review Pass (repeats per pass)
- **Trigger:** Each review pass completes
- **Writer-facing output:** Finding count by severity for this pass
- **Required input:** None (informational checkpoint)
- **Artifact written:** Findings appended to review artifact on disk (checkpoint/recovery)
- **Failure path:** If findings minimal (<3 total), note and consider skipping lower-priority passes

### Checkpoint 3: Issue Triage
- **Trigger:** All passes complete, Hold-severity findings exist
- **Writer-facing output:** Hold findings require acknowledgment before review closes
- **Required input:** Writer acknowledges each Hold finding (accept, dispute, or escalate)
- **Artifact written:** Triage decisions recorded in review artifact
- **Failure path:** Disputed findings marked "disputed" (not removed), excluded from summary count

### Checkpoint 4: Writer Resolution
- **Trigger:** After triage (or after passes if no Holds)
- **Writer-facing output:** Consolidated review report with options
- **Required input:** Writer chooses: done / another pass / target specific concern / escalate to writing-guide
- **Artifact written:** Final review artifact with resolution status
- **Failure path:** Escalation triggers Direction 3 payload to writing-guide

---

## 4. Dispatch Logic

Dispatch maps the writer's stated concern to the minimum effective module set, preventing feedback overload.

### Dispatch Logic
See `dispatch-and-escalation.md#concern-to-pass-dispatch-table` for the full concern-to-pass routing.

Concern tags: `sounds-too-ai`, `needs-tightening`, `rhythm-off`, `full-review`, `quick-polish`.

---

## 5. Edit Modes

Three compliant modes provide escalating levels of guidance while keeping the writer as author.

### Flag-only (default)
Identifies issue, names pattern, cites severity. No fix suggestion.
> Line 47: AI-ISM — hedging phrase. Severity: Flag.

### Flag-with-diagnostic
Adds WHY explanation referencing craft principles, voice profile, and genre norms.
> Line 47: AI-ISM — "it seems like perhaps this could indicate" is a hedging chain (4 qualifiers in 7 words). Your voice profile baseline is direct and declarative. This construction departs from that pattern and reads as machine-generated. Severity: Flag.

### Flag-with-structural-alternative
Describes the direction of a fix without providing replacement text.
> Line 47: AI-ISM — hedging chain. Your voice is direct. Fix direction: identify the specific claim this sentence is making and state it without qualification. If uncertainty is the point, name what's uncertain and why. Severity: Flag.

**Diagnostic creep anti-pattern:** Diagnostics must describe the *category* and *effect* of the problem, not the *specific textual correction*. If a diagnostic contains enough information to mechanically produce the fix without creative judgment, it has crossed from diagnosis into ghostwriting. Pull back.

**Redirect pattern:** When a writer says "just fix it" or "rewrite this for me":
> "I can flag what's not working and explain why, but the words are yours. Want me to run a [concern-appropriate] review so you can see exactly what to target?"

---

## 6. Severity Model (Dual-Axis)

### Severity (issue property)

See `review-protocol.md#severity-levels` for the full model (Hold / Flag / Advisory + Echo modifier).

### Intensity (delivery property)

| Level | Tone | When |
|---|---|---|
| **Gentle** | Reflective, observational | Writer chose gentle; early relationship |
| **Standard** | Direct naming + brief explanation | Default |
| **Firm** | Prioritized, consequence-aware | Writer chose firm; Workshop mode |
| **Diagnostic** | Stop-and-redirect, structural reframing | Premise-level problem detected |

### Cross-Axis Rules

| Issue Severity | Gentle | Standard | Firm | Diagnostic |
|---|---|---|---|---|
| Advisory | Noted lightly | Named briefly | Clear recommendation | Only if relevant to diagnosis |
| Flag | Surfaced as observation | Stated with impact | Stated with priority | Included in diagnostic framing |
| Hold | Surfaced clearly (never hidden) | Named with blocking status | Named with consequence chain | Triggers diagnostic intervention |

**Enforcement rules:**
- Holds are never delivered gently (minimum standard intensity)
- Advisories can be delivered at any intensity
- Diagnostics override the selected intensity — if a diagnostic-level problem is found, it's delivered at diagnostic intensity regardless of the writer's setting

---

## 7. Reference File Loading

prose-editor uses 4 reference files loaded on-demand per review phase.

### Loading Table

| Reference | Load When |
|---|---|
| `references/review-protocol.md` | Review start |
| `references/voice-profile-protocol.md` | Pass 1 or Voice Profile creation |
| `references/dispatch-and-escalation.md` | Intake (dispatch) or escalation |
| `references/lint/lint-ai-ism.md` | Pass 3, AI-ism sub-module active (~150 lines) |
| `references/lint/lint-sentence-variation.md` | Pass 3, variation active (~60 lines) |
| `references/lint/lint-diction.md` | Pass 3, diction active (~65 lines) |
| `references/lint/lint-cliche.md` | Pass 3, cliche active (~50 lines) |
| `references/lint/lint-echo.md` | Pass 3, echo active (~45 lines) |
| `references/lint/lint-readability.md` | Pass 3, readability active (~80 lines) |

Reference files load when their pass is active, not at session start.

**Sub-module selection:** For Pass 3, load only the sub-modules from `references/lint/` matching the dispatch concern tag. A `sounds-too-ai` review loads AI-ism Detection + Diction Audit sub-modules (~100 lines), not all six.

---

## 8. Review Artifact Schema

Review artifacts are written to `docs/writing/[project]/chapters/[nn]-[slug]-review.md`.

### Structured Header Fields

| Field | Type | Required | Notes |
|---|---|---|---|
| `project` | string | yes | Project slug |
| `chapter` | int | yes | Chapter number |
| `reviewed_at` | ISO date | yes | Review date |
| `entry_mode` | enum | yes | `context-only` or `post-handoff` |
| `concern` | enum | yes | Concern tag from intake |
| `edit_mode` | enum | yes | Selected edit mode |
| `intensity` | enum | yes | Selected intensity |
| `genre` | string | no | Genre tag or "unspecified" |
| `audience` | string | no | Audience or "unspecified" |
| `voice_profile` | string | no | Path or "none" |
| `passes_run` | int[] | yes | Pass numbers executed |
| `passes_requested` | int[] | yes | Pass numbers requested by dispatch |
| `last_completed_pass` | int | yes | For session resume |
| `handoff_version` | string | no | Handoff file version if post-handoff |
| `iteration` | int | yes | Review iteration number (1-3) |

### Freeform Body Template

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
[facts flagged for potential ledger entry — informational only]

### Pass Notes
[per-pass observations, skip declarations, checkpoint decisions, write confirmations]
```

Write confirmations follow `../../shared/write-protocol.md` -- single-line format after each write.

---

## 9. Anti-Ghostwriting Enforcement

prose-editor enforces the anti-ghostwriting boundary locally. No escalation pathway — the skill refuses the action directly.

### What IS permitted
- Naming the pattern: "This is a hedging chain"
- Explaining the principle: "Direct assertions read stronger than qualified ones"
- Describing the direction: "This passage would benefit from specificity over qualification"
- Referencing the voice profile: "Your baseline is [X]; this passage departs from it"
- Single-word category labels: "NOMINALIZATION", "AI-ISM", "ECHO"

### What is NEVER permitted
- Output replacement sentences, phrases, or words
- Complete unfinished sentences
- Rewrite passages (even if asked)
- Generate example "improved" versions of flagged text
- Produce before/after comparisons with AI-generated "after" text

### Diagnostic creep detection
If a finding contains enough information to mechanically produce the fix without creative judgment, it has crossed from diagnosis into ghostwriting. The signal: could the writer paste this finding into a search-and-replace and get a finished sentence? If yes, compress the diagnostic back to category + effect + direction.

---

## 10. Anti-Drift Controls

Two mechanisms prevent prose-editor from generating contradictory or unreliable findings.

### Cross-pass consistency check
Before writing findings from Pass N, compare against findings from Passes 1..N-1 for contradictions. If Pass 1 (voice) flags a passage as "too formal" but Pass 3 (prose quality) flags the same passage as "appropriately precise for the genre," surface the contradiction as a note in Pass Notes rather than silently presenting contradictory findings.

### Writer-disputed findings
When a writer challenges a finding during Issue Triage, the finding is marked "disputed" in the review artifact — not removed. Disputed findings are excluded from the summary's issue count but remain visible for reference. This preserves the diagnostic record while respecting the writer's judgment.

---

## 11. Session Resume

When writing-guide detects an incomplete review artifact, it offers session resume.

### 4-Step Decision Tree

1. **Detect:** Check for `[nn]-[slug]-review.md` in `docs/writing/[project]/chapters/` with `last_completed_pass` < total passes requested
2. **Load:** Read partial review artifact, extract completed findings and checkpoint state
3. **Present:** "Review in progress — [N] of [M] passes complete, [X] findings so far. Continue from Pass [N+1]?"
4. **Resume or restart:** Writer chooses to continue from next incomplete pass, or start fresh (new review artifact, iteration incremented)

### Resume Summary Template
```
## Session Resume
- Project: [slug]
- Chapter: [nn]-[slug]
- Passes completed: [list]
- Passes remaining: [list]
- Findings so far: N (X hold, Y flag, Z advisory)
- Last checkpoint: Pass [N] completed at [date]
```

---

## 12. Capabilities Not Yet Implemented

Future capabilities are documented in `docs/superpowers/plans/prose-editor-phase2-roadmap.md`. They are not loaded at runtime.
