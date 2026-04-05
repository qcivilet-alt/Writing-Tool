# Writing Tool — Deployment Migration, UX Fixes, and Schema Consistency

**Date:** 2026-04-05
**Status:** Draft
**Scope:** Plugin deployment structure, artifact write UX, field name consistency, code review cleanup

---

## 1. Executive Summary

The Writing Tool simplification (31 tasks, completed 2026-04-05) produced correct skill files but deployed them to the wrong location. A dry run revealed that Claude Code's Skill tool loads from `writing-tool-plugin/skills/` (via `.claude-plugin/plugin.json`), not from `.claude/skills/` where the simplified files reside. This spec addresses the deployment gap, adds a UX improvement discovered during the dry run, fixes pre-existing schema mismatches, and cleans up deferred code review items.

---

## 2. Deployment Migration (Plugin-First)

### Problem

Simplified skill files live in `.claude/skills/` (git-tracked in the worktree). The plugin system reads from `writing-tool-plugin/skills/` (untracked, contains old pre-simplification versions). The Skill tool loads the old versions at runtime.

### Solution

Move all simplified files into `writing-tool-plugin/`. Commit the plugin to git. Delete `.claude/skills/` and `.claude/shared/`.

### Post-Migration Structure

```
writing-tool-plugin/
├── .claude-plugin/plugin.json
├── shared/
│   ├── anti-ghostwriting.md
│   ├── write-protocol.md
│   └── escalation-schemas.md
└── skills/
    ├── story-architect/
    │   ├── SKILL.md
    │   └── references/
    │       ├── artifact-schemas.md
    │       ├── character-frames.md
    │       ├── friction-patterns.md
    │       ├── genre-vocabulary.md
    │       └── kb-gate-map.md
    ├── prose-editor/
    │   ├── SKILL.md
    │   └── references/
    │       ├── dispatch-and-escalation.md
    │       ├── review-protocol.md
    │       ├── voice-profile-protocol.md
    │       └── lint/
    │           ├── lint-ai-ism.md
    │           ├── lint-sentence-variation.md
    │           ├── lint-diction.md
    │           ├── lint-cliche.md
    │           ├── lint-echo.md
    │           └── lint-readability.md
    └── writing-guide/
        ├── SKILL.md
        └── references/
            ├── escalation-triage.md
            ├── output-patterns.md
            └── specialist-routing-guide.md
```

### Read Directive Path Resolution

From any skill directory (`writing-tool-plugin/skills/[module]/`):
- `../../shared/anti-ghostwriting.md` resolves to `writing-tool-plugin/shared/anti-ghostwriting.md`
- `references/friction-patterns.md` resolves to `writing-tool-plugin/skills/[module]/references/friction-patterns.md`

Same relative path pattern as `.claude/skills/` — no Read directives need updating.

### Steps

1. Delete old `writing-tool-plugin/skills/` contents (pre-simplification copies)
2. Copy `.claude/skills/*` into `writing-tool-plugin/skills/`
3. Copy `.claude/shared/*` into `writing-tool-plugin/shared/`
4. Delete `.claude/skills/` and `.claude/shared/`
5. `git add writing-tool-plugin/` — commit plugin to git
6. Verify Skill tool loads the new versions (restart session, invoke `/story-architect`)

### plugin.json Update

Update description to reflect prose-editor availability:

```json
{
  "name": "writing-tool",
  "description": "Author-led writing ecosystem: story planning (story-architect), editorial review (prose-editor), and craft mentorship (writing-guide). Supports fiction, memoir, CNF, and essay.",
  "version": "1.1.0"
}
```

---

## 3. Schema Consistency Fixes

### Problem

Field names in `kb-gate-map.md` (gate requirements) don't match field names in `artifact-schemas.md` (actual artifact templates) or the chapter file format.

### Mismatches

| Gate (kb-gate-map.md) | Current Name | Artifact (actual) | Fix |
|---|---|---|---|
| Chapter Direction | `entry_exit_beats` | `entry_state` + `exit_state` | Change gate map to list `entry_state`, `exit_state` |
| Chapter Direction | `point_of_view_confirmed` | `pov` | Change gate map to `pov` |
| Structure Approval | `inciting_incident_placed` | Used but not in manifest schema | Add to structure-map.md schema in artifact-schemas.md |

### Broken Section Anchors

The following anchors are referenced but don't exist as headings:

| Referenced Anchor | From | Fix |
|---|---|---|
| `kb-gate-map.md#intake-gate` | story-architect SKILL.md | Add `### Intake Gate` sub-heading in kb-gate-map.md |
| `kb-gate-map.md#premise-lock-gate` | story-architect SKILL.md | Add `### Premise Lock Gate` sub-heading |
| `kb-gate-map.md#character-commitment-gate` | story-architect SKILL.md | Add `### Character Commitment Gate` sub-heading |
| `kb-gate-map.md#structure-approval-gate` | story-architect SKILL.md | Add `### Structure Approval Gate` sub-heading |
| `kb-gate-map.md#chapter-direction-gate` | story-architect SKILL.md | Add `### Chapter Direction Gate` sub-heading |
| `kb-gate-map.md#handoff-gate` | story-architect SKILL.md | Add `### Handoff Gate` sub-heading |
| `dispatch-and-escalation.md#concern-to-pass-dispatch-table` | prose-editor SKILL.md | Add matching heading to dispatch-and-escalation.md |
| `artifact-schemas.md#escalation-payload-schemas` | story-architect SKILL.md | Update reference to `#escalation-format` (new heading) |

---

## 4. UX Improvement: Hybrid Write-on-Confirm

### Problem

During the dry run, the writer answered 7+ friction questions before anything was written to disk. If the session crashes mid-gate, all answers are lost.

### Solution

After each friction question answer passes validation, offer to save immediately:

```
Updated: core_tension -> "[value]"

Save to story-manifest.md? (y / continue without saving)
```

- "y" -> write immediately, confirm with single-line write protocol
- continue -> field stays in conversation, written at gate clearance
- Gate clearance always does a final batch write of any unsaved fields

### Implementation

Add step 5b to story-architect SKILL.md Section 5 (gate interaction flow):

```
5. Pass — field validated.
5b. Write checkpoint: show updated value, offer to save.
    Writer confirms → write + confirm.
    Writer defers → field held in conversation, written at gate clearance.
```

prose-editor is unaffected (it doesn't have incremental field writes).

---

## 5. Code Review Cleanup

### S1: Define [S] marker

Add to review-protocol.md severity section:
```
S = Source — modifier indicating the pattern has published research backing.
Not a severity level. Informational only.
```

### S2: Fix "write notices/receipts" in review artifact template

In prose-editor SKILL.md, replace "write notices/receipts" with "write confirmations" in the Pass Notes section of the review artifact template.

### I2: Broken anchors

Covered in Section 3 above.

---

## 6. Files Changed

### Files Modified

| File | Changes |
|---|---|
| `writing-tool-plugin/.claude-plugin/plugin.json` | Update description and version |
| `writing-tool-plugin/skills/story-architect/SKILL.md` | Add write checkpoint step (5b), fix escalation anchor ref |
| `writing-tool-plugin/skills/story-architect/references/kb-gate-map.md` | Fix 2 field names, add 6 gate sub-headings |
| `writing-tool-plugin/skills/story-architect/references/artifact-schemas.md` | Add inciting_incident_placed to structure-map schema |
| `writing-tool-plugin/skills/prose-editor/SKILL.md` | Fix "write notices/receipts" reference |
| `writing-tool-plugin/skills/prose-editor/references/review-protocol.md` | Define [S] marker |
| `writing-tool-plugin/skills/prose-editor/references/dispatch-and-escalation.md` | Add anchor heading for dispatch table |

### Files Created (in writing-tool-plugin/)

All files migrated from `.claude/skills/` and `.claude/shared/`. See Section 2 for full list.

### Files Deleted

| Path | Reason |
|---|---|
| `.claude/skills/` (entire directory) | Migrated to writing-tool-plugin/skills/ |
| `.claude/shared/` (entire directory) | Migrated to writing-tool-plugin/shared/ |

---

## 7. Verification

After implementation:

1. `ls writing-tool-plugin/skills/*/SKILL.md` — all 3 skills present
2. `ls writing-tool-plugin/shared/` — 3 shared files present
3. `ls .claude/skills/ .claude/shared/` — both deleted (should fail)
4. Restart Claude Code session, invoke `/story-architect` — verify simplified version loads (check for "clearance checkpoints" language, not "cannot be skipped")
5. Run the 8 verification checks from the simplification handoff against the new location
6. Grep for remaining broken anchors across `writing-tool-plugin/`
