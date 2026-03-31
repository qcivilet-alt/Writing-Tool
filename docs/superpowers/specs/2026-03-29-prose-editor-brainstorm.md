# prose-editor Integration Brainstorm
updated: 2026-03-29
scope: prose-editor module — integration strategy using Prose Editing Toolkit + Writing Pipeline
status: complete — all decisions confirmed

---

## Context

The Writing Tool is a three-module ecosystem: **writing-guide** (built, hub), **story-architect** (built, planning specialist), and **prose-editor** (planned, not yet built). Two external resources have been identified that directly address prose-editor's scope:

- **Claude Prose Editing Toolkit** — 9 reusable prompt modules for prose quality analysis (AI-ism detection, sentence variation, diction audit, cliche scanning, echo detection, readability/pacing, voice profiling, combined lint, post-edit verification)
- **Writing Pipeline Skill** — Structured intake-plan-execute-verify workflow for prose editing, with concern-based module dispatch and genre/audience calibration

This document maps how these resources integrate into the planned prose-editor module.

---

## 1. Alignment Map — Where Resources Fit

### Natural mappings

| prose-editor Need | Resource A (Toolkit) | Resource B (Pipeline) |
|---|---|---|
| Prose quality review passes | Modules 1-6 (AI-ism, rhythm, diction, cliches, echoes, readability) | Genre/audience calibration for each module |
| Voice consistency checking | Module 7 (Voice Profile) + Module 9 (Post-Edit Verification) | Stage 0 voice profile detection |
| Module dispatch logic | Module 8 (combined pass) vs individual modules | Stage 2 concern-to-module routing |
| Structured review workflow | — | Full intake-plan-execute-verify cycle |
| Output format / severity | 3-tier severity system (red/yellow/green) | Structured report format with summary |
| Iteration control | — | 3-pass over-editing limit with verify stage |
| Writer autonomy guardrails | "Flag only" default mode | "Never rewrite without being asked" guardrail |

### Gaps — what neither resource covers

1. **Continuity ledger** — tracking committed facts across chapters
2. **Structural alignment** — cross-referencing prose against story-architect planning artifacts
3. **Character consistency** — checking character behavior against committed frames
4. **Escalation payloads** — inter-skill communication with writing-guide/story-architect
5. **Handoff context loading** — consuming story-architect's `handoff-context.json`
6. **Intentional voice deviation** — letting writers mark passages as deliberate departures
7. **Context-only mode** — operating without planning artifacts when no handoff exists

---

## 2. Key Conflicts and Resolutions

### Conflict A: "Rewrite directly" mode vs. anti-ghostwriting guardrail

The toolkit offers "rewrite directly." The Writing Tool's anti-ghostwriting constraint is **non-bypassable**.

**Resolution:** Replace with three compliant modes:
1. **Flag-only** (default) — identify issue, name pattern, cite severity
2. **Flag-with-diagnostic** — add WHY explanation referencing craft principles and voice profile
3. **Flag-with-structural-alternative** — describe the shape of a fix without providing replacement text ("make this more direct" but never "change this to [new words]")

### Conflict B: Conversation-based workflow vs. file-based artifacts

The pipeline operates in conversation context. The Writing Tool uses file-based artifacts as ground truth.

**Resolution:** Pipeline stages map to artifact operations:
- Intake answers → stored as review artifact header
- Module findings → written to `chapters/[nn]-[slug]-review.md` with Write Notice/Receipt
- Verification → written to review artifact closure section
- Voice Profile → persisted at `docs/writing/[project]/voice-profile.md`

### Conflict C: Severity tiers vs. intensity modes

Toolkit uses 3-tier severity (issue property). Writing-guide uses 4-tier intensity (delivery property). These are different axes.

**Resolution:** Both coexist:
- **Severity** (Hold/Flag/Advisory) describes the issue's consequence
- **Intensity** (gentle/standard/firm/diagnostic) controls delivery tone
- A Hold is never delivered gently. An Advisory can be delivered firmly in Workshop mode.

| Issue Severity | Gentle | Standard | Firm | Diagnostic |
|---|---|---|---|---|
| Advisory | Noted lightly | Named briefly | Clear recommendation | Only if relevant to diagnosis |
| Flag | Surfaced as observation | Stated with impact | Stated with priority | Included in diagnostic framing |
| Hold | Surfaced clearly (holds are never hidden) | Named with blocking status | Named with consequence chain | Triggers diagnostic intervention |

---

## 3. Integration Architecture

### Where the toolkit modules live

NOT as KB leaf guides (those are craft guidance). Instead as **prose-editor reference files**:

```
~/.claude/skills/prose-editor/
  SKILL.md                              # ~400-480 lines
  references/
    prose-lint-modules.md               # Modules 1-6 consolidated, adapted
    voice-profile-protocol.md           # Module 7 adapted
    post-edit-verification.md           # Module 9 adapted
    module-dispatch-map.md              # Pipeline's concern-to-module routing
    review-protocol.md                  # Five-pass review logic
    escalation-payloads.md              # Inter-skill communication
```

### Two new KB leaf guides (for writing-guide routing)

```
docs/superpowers/writing-kb/writing_mentor_kb/02_leaf_guides/prose/
  GUIDE.PROSE.AI_PATTERN_DETECTION.md   # Distilled from Module 1
  GUIDE.PROSE.VOICE_CONSISTENCY.md      # Craft guide for voice drift
```

### Dispatch logic (in SKILL.md)

Adapted from Pipeline's Stage 2:
- "Sounds too AI" → AI-ism detection + diction audit + voice check
- "Needs tightening" → diction audit + cliche scanner + echo detection
- "Rhythm feels off" → sentence variation + echo detection + readability
- "Full review" → all five passes in order
- "Quick polish" → combined lint (compact), top 5 findings only

### Five-pass review protocol

1. **Structural alignment** — chapter vs. chapter plan vs. locked gates (Phase 2)
2. **Character consistency** — character behavior vs. committed frames (Phase 2)
3. **Voice coherence** — prose vs. voice profile + voice anchors (Modules 7, 9)
4. **Continuity check** — prose vs. continuity ledger; flag new facts (Phase 1: read-only)
5. **Prose quality** — AI-ism detection, diction, rhythm, echo, readability (Modules 1-6)

---

## 4. Voice Profile System

### Artifact location
`docs/writing/[project]/voice-profile.md` — project-level, not per-character

### Creation paths (both available)
- **Path A (during story-architect):** Minimal stub created at Character Commitment gate via friction questions about tone, formality, rhythm
- **Path B (at prose-editor first run):** Derived from submitted text, presented to writer for confirmation/modification

### Integration with handoff
Voice Profile fills the `voice_profile` gate key and `artifacts.voice_doc` field in `handoff-context.json`

---

## 5. Recommended Strategy — Staged Integration

### Phase 1: Editorial Core

Build prose-editor with prose quality passes and voice system. Immediate value for writers with prose in hand.

**Delivers:**
- SKILL.md with entry points, hard boundaries, dispatch logic, intensity mapping
- All reference files (prose-lint, voice-profile, post-edit-verification, dispatch map, escalation payloads)
- 2 KB leaf guides for writing-guide routing
- Context-only mode as primary mode
- Handoff loading (reads planning artifacts as context, no dedicated structural checks)
- Voice Profile creation (both paths) and comparison
- Read-only continuity ledger (flag candidate facts, don't write to ledger)
- Review artifacts to `chapters/[nn]-[slug]-review.md`
- Update specialist-routing-guide.md to activate prose-editor triggers

**Defers:**
- Structural alignment pass (chapter vs. chapter plan)
- Character consistency pass (behavior vs. committed frames)
- Continuity ledger read-write integration
- Voice deviation protocol
- Handoff version management

**Estimated:** ~8 new files, ~1,100 lines

### Phase 2: Structural Integration

Adds planning-aware passes — the differentiator from standalone lint tools.

**Delivers:**
- Structural alignment protocol
- Character consistency protocol
- Continuity ledger full read-write integration
- Voice deviation annotations
- Handoff version mismatch detection

**Estimated:** ~3 additional files, ~350 lines

---

## 6. What This Means for the Writer

**Writer WITH story-architect planning:**
1. Submit a chapter draft to prose-editor
2. Get structured review: voice consistency, prose quality, (Phase 2) structural alignment
3. Findings organized by severity with genre-calibrated thresholds
4. Revise and resubmit for post-edit verification (max 3 passes)
5. Voice profile respected throughout

**Writer WITHOUT story-architect:**
1. Paste any text and request a review
2. Intake questions: genre, audience, concern, edit mode
3. Targeted module analysis based on primary concern
4. Build voice profile from submitted text

---

## 7. Confirmed Decisions

1. **Continuity:** Phase 1 includes read-only continuity ledger. Full read-write deferred to Phase 2.
2. **Voice Profile:** Both creation paths available. Whichever the writer encounters first.
3. **Integration format:** Adapt and consolidate toolkit modules into purpose-built reference files. Modules 1-6 consolidated. Detection patterns preserved, structural logic adapted.
4. **Anti-ghostwriting:** Three compliant edit modes replace "rewrite directly." Non-negotiable hard boundary in SKILL.md.

---

## 8. Critical Files for Implementation

| File | Role |
|---|---|
| `docs/superpowers/specs/2026-03-28-writing-tool-design.md` | Design spec — constraints and open questions |
| `~/.claude/skills/story-architect/references/artifact-schemas.md` | Handoff and review artifact schemas |
| `~/.claude/skills/writing-guide/references/specialist-routing-guide.md` | Routing rules (needs update) |
| `~/.claude/skills/writing-guide/references/escalation-triage.md` | Escalation payload processing |
| `docs/superpowers/writing-kb/.../MODE.INTENSITY_CONTROL.md` | Intensity modes prose-editor must respect |

---

## 9. Open Questions for prose-editor Design Spec

These should be resolved during the prose-editor design spec session (next task):

- **OQ-PE1:** How does the read-only continuity ledger present flagged facts to the writer? Inline in the review artifact, or as a separate "candidate facts" appendix?
- **OQ-PE2:** Should the Voice Profile stub (Path A) include voice anchors from character files, or keep those separate?
- **OQ-PE3:** What is the minimum text length for meaningful prose-editor analysis? (The toolkit works on any length, but structural passes need critical mass.)
- **OQ-PE4:** How should prose-editor handle multi-POV chapters where different sections may have intentionally different voice characteristics?
