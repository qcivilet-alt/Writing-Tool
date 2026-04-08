# Deployment Migration, UX Fixes, and Schema Consistency — Implementation Plan

> **For agentic workers:** REQUIRED: Use superpowers:subagent-driven-development (if subagents available) or superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Move simplified skill files from `.claude/skills/` to `writing-tool-plugin/skills/`, fix schema mismatches, add hybrid write-on-confirm UX, and clean up deferred code review items.

**Architecture:** All changes are markdown file edits and directory operations. No compiled code. The plugin directory (`writing-tool-plugin/`) becomes the single canonical location for all skill files, tracked in git.

**Tech Stack:** Markdown files, YAML frontmatter, Claude Code plugin system, git.

**Spec:** `docs/superpowers/specs/2026-04-05-deployment-and-ux-fixes.md`

---

## File Map

### Files to CREATE (new plugin structure)

| File | Source |
|---|---|
| `writing-tool-plugin/.claude-plugin/plugin.json` | Updated from main tree version |
| `writing-tool-plugin/shared/anti-ghostwriting.md` | Copy from `.claude/shared/` |
| `writing-tool-plugin/shared/write-protocol.md` | Copy from `.claude/shared/` |
| `writing-tool-plugin/shared/escalation-schemas.md` | Copy from `.claude/shared/` |
| `writing-tool-plugin/skills/story-architect/SKILL.md` | Copy from `.claude/skills/` |
| `writing-tool-plugin/skills/story-architect/references/*.md` (5 files) | Copy from `.claude/skills/` |
| `writing-tool-plugin/skills/prose-editor/SKILL.md` | Copy from `.claude/skills/` |
| `writing-tool-plugin/skills/prose-editor/references/*.md` (3 files) | Copy from `.claude/skills/` |
| `writing-tool-plugin/skills/prose-editor/references/lint/*.md` (6 files) | Copy from `.claude/skills/` |
| `writing-tool-plugin/skills/writing-guide/SKILL.md` | Copy from `.claude/skills/` |
| `writing-tool-plugin/skills/writing-guide/references/*.md` (3 files) | Copy from `.claude/skills/` |

### Files to MODIFY (after copy)

| File | Changes |
|---|---|
| `writing-tool-plugin/skills/story-architect/SKILL.md` | Add write checkpoint step (5b) |
| `writing-tool-plugin/skills/story-architect/references/kb-gate-map.md` | Fix 2 field names, add 6 gate sub-headings |
| `writing-tool-plugin/skills/story-architect/references/artifact-schemas.md` | Rename escalation heading, add inciting_incident_placed to structure-map schema |
| `writing-tool-plugin/skills/prose-editor/SKILL.md` | Fix "write notices/receipts" reference |
| `writing-tool-plugin/skills/prose-editor/references/review-protocol.md` | Define [S] marker |
| `writing-tool-plugin/skills/prose-editor/references/dispatch-and-escalation.md` | Add anchor heading for dispatch table |

### Files to DELETE

| Path | Reason |
|---|---|
| `.claude/skills/` (entire directory) | Migrated to writing-tool-plugin/ |
| `.claude/shared/` (entire directory) | Migrated to writing-tool-plugin/ |

---

## Chunk 1: Deployment Migration (Tasks 1-2)

### Task 1: Create plugin directory structure and copy files

**Files:**
- Create: `writing-tool-plugin/` (full tree as specified in spec Section 2)

- [ ] **Step 1: Create plugin directory structure**

```bash
cd "C:/Users/qcivi/OneDrive/Documents/AI/Cursor/Writing Tool/.claude/worktrees/naughty-almeida"
mkdir -p writing-tool-plugin/.claude-plugin
mkdir -p writing-tool-plugin/shared
mkdir -p writing-tool-plugin/skills/story-architect/references
mkdir -p writing-tool-plugin/skills/prose-editor/references/lint
mkdir -p writing-tool-plugin/skills/writing-guide/references
```

- [ ] **Step 2: Create plugin.json**

Write `writing-tool-plugin/.claude-plugin/plugin.json`:

```json
{
  "name": "writing-tool",
  "description": "Author-led writing ecosystem: story planning (story-architect), editorial review (prose-editor), and craft mentorship (writing-guide). Supports fiction, memoir, CNF, and essay.",
  "version": "1.1.0",
  "author": { "name": "qcivilet-alt" },
  "repository": "https://github.com/qcivilet-alt/Writing-Tool"
}
```

- [ ] **Step 3: Copy shared files**

```bash
cp .claude/shared/anti-ghostwriting.md writing-tool-plugin/shared/
cp .claude/shared/write-protocol.md writing-tool-plugin/shared/
cp .claude/shared/escalation-schemas.md writing-tool-plugin/shared/
```

- [ ] **Step 4: Copy story-architect files**

```bash
cp .claude/skills/story-architect/SKILL.md writing-tool-plugin/skills/story-architect/
cp .claude/skills/story-architect/references/*.md writing-tool-plugin/skills/story-architect/references/
```

- [ ] **Step 5: Copy prose-editor files**

```bash
cp .claude/skills/prose-editor/SKILL.md writing-tool-plugin/skills/prose-editor/
cp .claude/skills/prose-editor/references/*.md writing-tool-plugin/skills/prose-editor/references/
cp .claude/skills/prose-editor/references/lint/*.md writing-tool-plugin/skills/prose-editor/references/lint/
```

- [ ] **Step 6: Copy writing-guide files**

```bash
cp .claude/skills/writing-guide/SKILL.md writing-tool-plugin/skills/writing-guide/
cp .claude/skills/writing-guide/references/*.md writing-tool-plugin/skills/writing-guide/references/
```

- [ ] **Step 7: Verify all 22 files copied**

```bash
find writing-tool-plugin -type f | wc -l
# Expected: 23 (22 skill/shared files + 1 plugin.json)
find writing-tool-plugin -type f | sort
```

- [ ] **Step 8: Verify Read directive paths resolve from new location**

```bash
# From each skill dir, check that ../../shared/anti-ghostwriting.md resolves
ls writing-tool-plugin/skills/story-architect/../../shared/anti-ghostwriting.md
ls writing-tool-plugin/skills/prose-editor/../../shared/anti-ghostwriting.md
ls writing-tool-plugin/skills/writing-guide/../../shared/anti-ghostwriting.md
# All three should succeed
```

- [ ] **Step 9: Commit migration**

```bash
git add writing-tool-plugin/
git commit -m "feat: migrate skills to writing-tool-plugin/ for plugin-first deployment"
```

---

### Task 2: Delete old skill locations

**Files:**
- Delete: `.claude/skills/` (entire directory)
- Delete: `.claude/shared/` (entire directory)

- [ ] **Step 1: Delete old directories**

```bash
rm -rf .claude/skills .claude/shared
```

- [ ] **Step 2: Verify deletion**

```bash
ls .claude/skills 2>&1  # Expected: No such file or directory
ls .claude/shared 2>&1  # Expected: No such file or directory
```

- [ ] **Step 3: Verify .claude/ still has settings**

```bash
ls .claude/
# Expected: settings.local.json (and possibly worktrees/)
```

- [ ] **Step 4: Commit deletion**

```bash
git rm -r .claude/skills .claude/shared
git commit -m "chore: delete old .claude/skills/ and .claude/shared/ (migrated to writing-tool-plugin/)"
```

Note: The `rm -rf` in Step 1 removes the files from disk. The `git rm -r` in Step 4 stages the deletion for commit. If the files were already removed from disk, `git rm -r` will still work (it stages the removal from the index).

---

## Chunk 2: Schema Consistency Fixes (Tasks 3-5)

### Task 3: Fix field name mismatches in kb-gate-map.md

**Files:**
- Modify: `writing-tool-plugin/skills/story-architect/references/kb-gate-map.md`

- [ ] **Step 1: Fix Chapter Direction field names**

In the Chapter Direction required fields table, replace:

| Old | New |
|---|---|
| `point_of_view_confirmed` | `pov` |
| `entry_exit_beats` | `entry_state`, `exit_state` |

Apply to all 4 genre rows in the Chapter Direction table.

Also in Section 4 (Gap Severity Assignments), replace `point_of_view_confirmed` with `pov` in the Critical severity examples column.

- [ ] **Step 2: Verify no remaining old field names**

```bash
grep -n "entry_exit_beats\|point_of_view_confirmed" writing-tool-plugin/skills/story-architect/references/kb-gate-map.md
# Expected: 0 matches
```

- [ ] **Step 3: Commit**

```bash
git add writing-tool-plugin/skills/story-architect/references/kb-gate-map.md
git commit -m "fix: align gate field names with artifact schemas (pov, entry_state/exit_state)"
```

---

### Task 4: Add gate sub-headings for anchor resolution

**Files:**
- Modify: `writing-tool-plugin/skills/story-architect/references/kb-gate-map.md`

- [ ] **Step 1: Rename existing gate headings and add missing ones in Section 3**

In Section 3 (Required Field Sets Per Gate x Genre), rename the four existing headings to add "Gate" suffix:

- `### Premise Lock` → `### Premise Lock Gate`
- `### Character Commitment` → `### Character Commitment Gate`
- `### Structure Approval` → `### Structure Approval Gate`
- `### Chapter Direction` → `### Chapter Direction Gate`

Add two new headings that don't currently exist:

Before `### Premise Lock Gate`, add:
```
### Intake Gate
Intake confirms: genre tag, working title, premise sketch, protagonist sketch. No required-field table — these are collected conversationally.
```

After `### Chapter Direction Gate`, add:
```
### Handoff Gate
Requires all prior gates cleared, no Critical-severity unresolved annotations.
```

- [ ] **Step 2: Verify anchors resolve**

```bash
grep -n "### .*Gate" writing-tool-plugin/skills/story-architect/references/kb-gate-map.md
# Expected: 6 matches (Intake, Premise Lock, Character Commitment, Structure Approval, Chapter Direction, Handoff)
```

- [ ] **Step 3: Commit**

```bash
git add writing-tool-plugin/skills/story-architect/references/kb-gate-map.md
git commit -m "fix: add gate sub-headings for anchor resolution from SKILL.md"
```

---

### Task 5: Fix escalation heading and dispatch table anchor

**Files:**
- Modify: `writing-tool-plugin/skills/story-architect/references/artifact-schemas.md` (escalation heading rename + add inciting_incident_placed to structure-map schema)
- Modify: `writing-tool-plugin/skills/prose-editor/references/dispatch-and-escalation.md`

- [ ] **Step 1: Rename escalation heading in artifact-schemas.md**

Change line with `## Escalation Format` back to `## Escalation Payload Schemas` to match the ToC entry at line 20 and the SKILL.md references at lines 63, 258.

- [ ] **Step 2: Add inciting_incident_placed to structure-map schema**

In artifact-schemas.md's structure-map.md Schema section, add to the field table:

```
| `inciting_incident_placed` | string | fiction, memoir | Location of the inciting incident in the act structure. |
```

- [ ] **Step 3: Add dispatch table anchor in dispatch-and-escalation.md**

Add `### Concern-to-Pass Dispatch Table` as a heading before the existing dispatch table (currently the table sits under `### Concern-to-Pass-to-Sub-Module Routing`). Either rename the existing heading or add a second anchor heading.

- [ ] **Step 4: Verify anchors**

```bash
grep -n "Escalation Payload Schemas" writing-tool-plugin/skills/story-architect/references/artifact-schemas.md
# Expected: matches in ToC and heading

grep -n "Concern-to-Pass" writing-tool-plugin/skills/prose-editor/references/dispatch-and-escalation.md
# Expected: heading present
```

- [ ] **Step 5: Commit**

```bash
git add writing-tool-plugin/skills/story-architect/references/artifact-schemas.md writing-tool-plugin/skills/prose-editor/references/dispatch-and-escalation.md
git commit -m "fix: align section anchors with cross-file references"
```

---

## Chunk 3: UX and Cleanup (Tasks 6-7)

### Task 6: Add hybrid write-on-confirm to gate interaction flow

**Files:**
- Modify: `writing-tool-plugin/skills/story-architect/SKILL.md`

- [ ] **Step 1: Read current Section 5 gate interaction flow**

Find the numbered interaction flow (steps 1-10). Identify step 5 ("**Pass** — field updated in artifact, next gap scanned, next question fires.") and step 6 ("**Fail** — answer validation response explains what's missing...").

- [ ] **Step 2: Add step 5b after step 5**

After step 5 ("Pass — field validated."), insert:

```markdown
5b. **Write checkpoint:** Show updated value, offer to save.
    Writer confirms → write field to artifact + single-line confirmation.
    Writer defers → field held in conversation, written at gate clearance.
```

- [ ] **Step 3: Verify insertion**

```bash
grep -n "Write checkpoint" writing-tool-plugin/skills/story-architect/SKILL.md
# Expected: 1 match
```

- [ ] **Step 4: Commit**

```bash
git add writing-tool-plugin/skills/story-architect/SKILL.md
git commit -m "feat: add hybrid write-on-confirm to story-architect gate flow"
```

---

### Task 7: Code review cleanup (S1, S2)

**Files:**
- Modify: `writing-tool-plugin/skills/prose-editor/references/review-protocol.md`
- Modify: `writing-tool-plugin/skills/prose-editor/SKILL.md`

- [ ] **Step 1: Define [S] marker in review-protocol.md**

After the Echo severity definition in the Severity Levels table/section, add:

```markdown
S = Source — modifier indicating the pattern has published research backing.
Not a severity level. Informational only. Example: `[A][S]` = advisory finding backed by published research.
```

- [ ] **Step 2: Fix "write notices/receipts" in prose-editor SKILL.md**

Find line ~279 containing `write notices/receipts` and replace with `write confirmations`.

- [ ] **Step 3: Verify**

```bash
grep -n "write notices/receipts" writing-tool-plugin/skills/prose-editor/SKILL.md
# Expected: 0 matches

grep -n "S = Source" writing-tool-plugin/skills/prose-editor/references/review-protocol.md
# Expected: 1 match
```

- [ ] **Step 4: Commit**

```bash
git add writing-tool-plugin/skills/prose-editor/SKILL.md writing-tool-plugin/skills/prose-editor/references/review-protocol.md
git commit -m "fix: define [S] marker, fix stale write-notice reference"
```

---

## Verification

After all tasks complete, run these checks:

```bash
ROOT="C:/Users/qcivi/OneDrive/Documents/AI/Cursor/Writing Tool/.claude/worktrees/naughty-almeida"

# 1. All 3 skills present
ls $ROOT/writing-tool-plugin/skills/*/SKILL.md

# 2. Shared files present
ls $ROOT/writing-tool-plugin/shared/

# 3. Old locations deleted
ls $ROOT/.claude/skills 2>&1   # should fail
ls $ROOT/.claude/shared 2>&1   # should fail

# 4. plugin.json valid
python -c "import json; json.load(open('$ROOT/writing-tool-plugin/.claude-plugin/plugin.json'))"

# 5. Read paths resolve
ls $ROOT/writing-tool-plugin/skills/story-architect/../../shared/anti-ghostwriting.md
ls $ROOT/writing-tool-plugin/skills/prose-editor/../../shared/anti-ghostwriting.md
ls $ROOT/writing-tool-plugin/skills/writing-guide/../../shared/anti-ghostwriting.md

# 6. No old field names
grep -rn "entry_exit_beats\|point_of_view_confirmed" $ROOT/writing-tool-plugin/skills/

# 7. Gate anchors exist
grep -c "### .*Gate" $ROOT/writing-tool-plugin/skills/story-architect/references/kb-gate-map.md
# Expected: 6

# 8. No old directory
ls $ROOT/writing-tool-plugin/skills/writing-guide-routing 2>&1  # should fail

# 9. No stale references
grep -rn "write notices/receipts" $ROOT/writing-tool-plugin/skills/
# Expected: 0 matches

# 10. Escalation heading aligned
grep "Escalation Payload Schemas" $ROOT/writing-tool-plugin/skills/story-architect/references/artifact-schemas.md | wc -l
# Expected: 2 (ToC + heading)
```

**Deferred to manual post-merge verification:** Runtime invocation tests (`/story-architect`, `/prose-editor`) require a Claude Code session restart and cannot be automated within this plan. Run these manually after merging.
