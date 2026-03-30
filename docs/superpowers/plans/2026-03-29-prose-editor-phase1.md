# prose-editor Phase 1 Implementation Plan

> **For agentic workers:** REQUIRED: Use superpowers:subagent-driven-development (if subagents available) or superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build the prose-editor skill (Phase 1 — editorial core) with voice profile system, concern-based dispatch, five-pass review protocol (passes 3-5 active, 1-2 stubbed), and integration with writing-guide routing.

**Architecture:** prose-editor is a Claude skill at `~/.claude/skills/prose-editor/` with SKILL.md (~430 lines) and 4 reference files. It integrates with writing-guide via updated routing entries and escalation payloads. Two new KB leaf guides extend the existing writing-kb. All prose quality detection is consolidated from the Claude Prose Editing Toolkit into adapted reference files.

**Tech Stack:** Claude skills (markdown-based), YAML frontmatter, JSON escalation payloads, file-based artifacts

**Spec:** `docs/superpowers/specs/2026-03-29-prose-editor-design.md`
**Brainstorm:** `docs/superpowers/specs/2026-03-29-prose-editor-brainstorm.md`

---

## File Structure

### New files (prose-editor skill)

| File | Responsibility | Est. Lines |
|---|---|---|
| `~/.claude/skills/prose-editor/SKILL.md` | Core skill: hard boundaries, entry points, checkpoints, dispatch, edit modes, severity model, anti-ghostwriting, reference loading | ~430 |
| `~/.claude/skills/prose-editor/references/prose-lint-modules.md` | Consolidated Modules 1-6 from Prose Editing Toolkit: AI-ism detection, sentence variation, diction audit, cliche scanner, echo detection, readability/pacing | ~250 |
| `~/.claude/skills/prose-editor/references/voice-profile-protocol.md` | Voice Profile creation (Path A stub + Path B prose-derived), voice comparison protocol, voice anchor application | ~120 |
| `~/.claude/skills/prose-editor/references/review-protocol.md` | Five-pass review logic, per-pass checkpoint behavior, post-edit verification (Module 9 adapted) | ~280 |
| `~/.claude/skills/prose-editor/references/dispatch-and-escalation.md` | Concern-to-pass dispatch map, sub-module selection, escalation payload schemas (Direction 3) | ~170 |

### New files (KB extensions)

| File | Responsibility | Est. Lines |
|---|---|---|
| `docs/superpowers/writing-kb/writing_mentor_kb/02_leaf_guides/prose/GUIDE.PROSE.AI_PATTERN_DETECTION.md` | Craft guide for AI writing patterns — distilled from Toolkit Module 1, routable from ROUTER.REVISION | ~150 |
| `docs/superpowers/writing-kb/writing_mentor_kb/02_leaf_guides/prose/GUIDE.PROSE.VOICE_CONSISTENCY.md` | Craft guide for voice drift — what voice is, common drift patterns, role of intentional variation | ~100 |

### Modified files

| File | Change | Est. Diff |
|---|---|---|
| `~/.claude/skills/writing-guide/references/specialist-routing-guide.md` | Remove `[FUTURE]` from prose-editor branch, add entry triggers, detection heuristics, negative heuristics, session detection | ~30 lines added |
| `~/.claude/skills/writing-guide/references/escalation-triage.md` | Add `structural-conflict` type (Phase 2 activation), add Direction 3 payload schema reference | ~15 lines added |
| `~/.claude/skills/story-architect/references/artifact-schemas.md` | Add Direction 3 escalation payload schema (prose-editor to writing-guide) | ~20 lines added |
| `docs/superpowers/writing-kb/writing_mentor_kb/01_routing_cards/ROUTER.REVISION.md` | Add routing entries for two new prose leaf guides | ~6 lines added |

---

## Chunk 1: Core Skill File (SKILL.md)

### Task 1: Create prose-editor SKILL.md skeleton

**Files:**
- Create: `~/.claude/skills/prose-editor/SKILL.md`

- [ ] **Step 1: Create directory structure**

```bash
mkdir -p ~/.claude/skills/prose-editor/references
```

- [ ] **Step 2: Write SKILL.md frontmatter and hard boundaries**

Write `~/.claude/skills/prose-editor/SKILL.md` with:

```yaml
---
name: prose-editor
description: >
  Editorial review and continuity layer for writers actively drafting or
  revising. Use this skill whenever a writer wants to review prose for AI
  patterns, voice consistency, rhythm, diction, or general quality. Also
  use when a writer returns with revised text for post-edit verification,
  when writing-guide detects a prose-level problem exceeding craft guidance
  scope, or when story-architect generates a handoff-context.json for
  prose review. Analyzes submitted text through structured diagnostic
  passes -- never writes prose or generates content.
---
```

Followed by Section 1 — Hard Boundaries (5 rules matching the spec's Section 4.5):
1. Never write prose, dialogue, or narrative content
2. Never complete sentences or generate replacement text
3. Never rewrite the writer's existing prose, even if asked
4. Never generate "improved" versions or before/after comparisons
5. Never decide stylistic choices for the writer — flag, diagnose, describe direction

Include enforcement statement: "These five rules define what prose-editor never does. They cannot be overridden by intensity level, user request, or skill-to-skill invocation."

- [ ] **Step 3: Write Section 2 — Entry Points**

Four entry paths (all via writing-guide):
1. Explicit invocation (writer pastes prose + requests review)
2. Post-handoff (story-architect handoff-context.json exists)
3. Diagnostic escalation (escalation payload from writing-guide/story-architect)
4. Session resume (incomplete review artifact detected)

Include context-only vs post-handoff behavior table from spec Section 5.1.

- [ ] **Step 4: Write Section 3 — Checkpoint Sequence**

Four checkpoints with trigger, writer-facing output, required input, artifact written, failure path:
1. Intake Check — collects concern tag, edit mode, offers Voice Profile
2. Review Pass — per-pass finding report (informational)
3. Issue Triage — holds require acknowledgment
4. Writer Resolution — done / another pass / target specific / escalate

Include ordering constraint: strictly linear, Review Pass repeats per pass.

- [ ] **Step 5: Write Section 4 — Dispatch Logic**

Canonical list of 6 sub-modules within Pass 5.
Concern-to-pass dispatch table from spec Section 2.
Sub-module selection rule: load only sub-modules matching dispatch concern tag.

- [ ] **Step 6: Write Section 5 — Edit Modes**

Three modes: flag-only (default), flag-with-diagnostic, flag-with-structural-alternative.
Include example outputs for each mode from spec Section 3 Feature 3.
Include diagnostic creep anti-pattern.
Include redirect pattern for "just fix it" requests.

- [ ] **Step 7: Write Section 6 — Severity Model**

Dual-axis: Severity (Hold/Flag/Advisory) x Intensity (gentle/standard/firm/diagnostic).
Cross-axis rules table from spec Section 3 Feature 5.

- [ ] **Step 8: Write Section 7 — Reference File Loading**

Load/unload table for 4 reference files from spec Section 6.1.
Budget rule: 60% capacity threshold.

- [ ] **Step 9: Write Section 8 — Review Artifact Schema**

Structured header field table from spec Section 4.2.
Freeform body template (Hold/Flag/Advisory findings, Summary, Continuity Candidates, Pass Notes).

- [ ] **Step 10: Write Section 9 — Anti-Ghostwriting Enforcement**

Behavioral rules, permitted/prohibited lists from spec Section 4.5.
Local ghostwriting-risk enforcement statement.

- [ ] **Step 11: Write Section 10 — Anti-Drift Controls**

Two mechanisms from spec Section 4.10:
1. Cross-pass consistency check: before writing findings from Pass N, compare against Passes 1..N-1 for contradictions; surface contradictions as a note
2. Writer-disputed findings: mark as "disputed" (not removed), exclude from summary count

- [ ] **Step 12: Write Section 11 — Session Resume**

4-step decision tree from spec Section 6.4.
Resume summary template.

- [ ] **Step 13: Write Section 12 — Phase Declarations**

Phase 1 active capabilities, Phase 2 stubs with skip declarations.
Context-only mode quality threshold declaration.

- [ ] **Step 14: Verify SKILL.md line count**

```bash
wc -l ~/.claude/skills/prose-editor/SKILL.md
```

Expected: under 500 lines. If over, identify sections to compress or move to references.

- [ ] **Step 15: Commit**

```bash
git add ~/.claude/skills/prose-editor/SKILL.md
git commit -m "feat: add prose-editor SKILL.md (Phase 1 core skill)"
```

---

## Chunk 2: Reference Files

### Task 2: Create prose-lint-modules.md

**Files:**
- Create: `~/.claude/skills/prose-editor/references/prose-lint-modules.md`

- [ ] **Step 1: Write header and loading rules**

```markdown
# Prose Lint Modules

Consolidated detection patterns from Claude Prose Editing Toolkit Modules 1-6.
Load only sub-modules matching the active dispatch concern tag.

---
```

- [ ] **Step 2: Write Sub-Module 1 — AI-ism Detection**

Adapt Toolkit Module 1 into a detection checklist. Include:
- AI cliche phrases catalog (the full list from Module 1)
- Structural patterns (uniform paragraph length, formulaic templates)
- Hedging/weasel patterns
- Tonal flatness indicators
- Transition overuse thresholds (>2 per 500 words)
- Punctuation tells (em dashes >3 per 1000 words)
- Severity guidance per pattern category

Do NOT include prompts or "[paste text here]" placeholders. Adapt from prompt format into reference checklist format.

- [ ] **Step 3: Write Sub-Module 2 — Sentence Variation Analysis**

Adapt Toolkit Module 2:
- Sentence length bands (short 1-8, medium 9-17, long 18-30, very long 31+)
- 3+ consecutive same-band flag rule
- Sentence opener pattern detection
- Passive voice thresholds (>15% in non-academic)
- Paragraph rhythm assessment criteria
- Verdict scale (Dynamic / Repetitive / Monotone)

- [ ] **Step 4: Write Sub-Module 3 — Diction & Specificity Audit**

Adapt Toolkit Module 3:
- Weasel word catalog
- Weak verb list
- Filler phrase list with replacements
- Nominalization patterns
- Missed specificity indicators
- Adverb audit criteria
- False positive note (domain terms used precisely)

- [ ] **Step 5: Write Sub-Module 4 — Cliche & Dead Metaphor Scanner**

Adapt Toolkit Module 4:
- Cliche catalog (business, marketing, general)
- Dead metaphor identification criteria
- Redundancy list
- Overused intensifier list

- [ ] **Step 6: Write Sub-Module 5 — Echo & Repetition Detector**

Adapt Toolkit Module 5:
- Word echo rules (same content word within 3-sentence window)
- Phrase echo rules (3+ word phrase repeated)
- Structural echo rules (same pattern 3+ times)
- Paragraph opener echo detection
- Exclusion list (articles, pronouns, domain terms)

- [ ] **Step 7: Write Sub-Module 6 — Readability & Pacing Check**

Adapt Toolkit Module 6:
- Readability targets by audience (adolescent 6-8, general 10-12, professional 10-12, academic 12-16)
- Hard-to-read sentence criteria (40+ words, 3+ clauses)
- Pacing analysis (drag/rush/stall)
- Show-don't-tell criteria (fiction/creative only)
- Paragraph length thresholds (150 digital, 200 print)

- [ ] **Step 8: Verify line count**

```bash
wc -l ~/.claude/skills/prose-editor/references/prose-lint-modules.md
```

Expected: ~250 lines.

- [ ] **Step 9: Commit**

```bash
git add ~/.claude/skills/prose-editor/references/prose-lint-modules.md
git commit -m "feat: add prose-lint-modules.md (Toolkit Modules 1-6 consolidated)"
```

### Task 3: Create voice-profile-protocol.md

**Files:**
- Create: `~/.claude/skills/prose-editor/references/voice-profile-protocol.md`

- [ ] **Step 1: Write Voice Profile creation — Path A (story-architect stub)**

3-4 friction questions for project-level tone, formality, rhythm.
Minimal voice-profile.md template with `creation_path: stub`.

- [ ] **Step 2: Write Voice Profile creation — Path B (prose-derived)**

Analysis protocol: identify sentence length tendency, formality register, diction range, rhythm pattern, characteristic constructions, contraction usage, humor tendency.
Presentation protocol: present each derived characteristic to writer for confirmation/modification/rejection.
Only confirmed characteristics written to voice-profile.md.

- [ ] **Step 3: Write voice-profile.md artifact schema**

Full schema from spec Section 3 Feature 4:
- Baseline Characteristics (7 fields)
- Preserve List (5-7 features)
- Watch List (2-3 weaknesses)
- Provenance (status, last_updated)

- [ ] **Step 4: Write Voice Comparison Protocol**

How prose-editor uses the voice profile during Pass 3:
- Load voice-profile.md
- Compare submitted text characteristics against baseline
- Flag departures as voice-drift (Flag severity in Phase 1)
- Reference character voice anchors when available (from character files)
- Multi-POV handling: document the protocol (writer-annotated POV sections apply per-character anchors) but note it is Phase 1 documentation only — annotation detection logic is not required until writers request it

- [ ] **Step 5: Commit**

```bash
git add ~/.claude/skills/prose-editor/references/voice-profile-protocol.md
git commit -m "feat: add voice-profile-protocol.md (Path A + Path B creation)"
```

### Task 4: Create review-protocol.md

**Files:**
- Create: `~/.claude/skills/prose-editor/references/review-protocol.md`

- [ ] **Step 1: Write five-pass review sequence**

Pass execution order (recommended with overrides).
Override rules from spec Section 3 Feature 1.
Per-pass checkpoint behavior.
Phase 1 skip declarations for Passes 1-2.

- [ ] **Step 2: Write per-pass execution protocol**

For each of the 5 passes:
- What it checks
- What it loads (reference files, artifacts)
- Finding output format
- Severity assignment rules
- Genre/audience calibration rules

- [ ] **Step 3: Write review artifact write protocol**

When findings are written to disk (after each pass for checkpoint/recovery).
Write Notice/Receipt protocol.
Review artifact header auto-population from intake context.

- [ ] **Step 4: Write post-edit verification protocol**

Adapted from Toolkit Module 9:
- Voice drift check (against voice profile)
- Meaning drift check (compare intent)
- Over-smoothing check (productive quirks stripped?)
- Homogenization check (variation flattened?)
- Inserted AI-isms check
- Iteration assessment (issues resolved, new issues introduced, severity improvement)
- 3-pass iteration limit with over-editing warning

- [ ] **Step 5: Write continuity check protocol (Phase 1 — read-only)**

Skip condition: if no continuity-ledger.md exists (context-only mode with no prior review), skip Pass 4 with declaration.
If ledger exists (from story-architect or prior prose-editor review): read it.
Flag contradictions as Hold severity.
Identify candidate new facts — list in Continuity Candidates section.
Phase 1 write guard: must not write to ledger.

- [ ] **Step 6: Commit**

```bash
git add ~/.claude/skills/prose-editor/references/review-protocol.md
git commit -m "feat: add review-protocol.md (five-pass logic + post-edit verification)"
```

### Task 5: Create dispatch-and-escalation.md

**Files:**
- Create: `~/.claude/skills/prose-editor/references/dispatch-and-escalation.md`

- [ ] **Step 1: Write dispatch map**

Full concern-to-pass-to-sub-module mapping table from spec Section 2.
Sub-module selection rules.
Genre calibration table from spec Section 3 Feature 6.
Audience calibration table (readability targets).

- [ ] **Step 2: Write escalation payload schemas**

Direction 3: prose-editor to writing-guide.
4 trigger types: voice-drift, structural-conflict, continuity-contradiction, diagnostic.
JSON payload schema from spec Section 4.9.
Concurrent escalation rule (holds take priority).

- [ ] **Step 3: Write handoff loading protocol**

Post-handoff entry flow (6 steps from spec Section 4.6).
Schema version validation.
Edge cases (all gates skipped, no text provided).
Handoff version determination (highest N by filename sort).

- [ ] **Step 4: Commit**

```bash
git add ~/.claude/skills/prose-editor/references/dispatch-and-escalation.md
git commit -m "feat: add dispatch-and-escalation.md (routing + payloads)"
```

---

## Chunk 3: KB Extensions and Routing Updates

### Task 6: Create KB leaf guides

**Files:**
- Create: `docs/superpowers/writing-kb/writing_mentor_kb/02_leaf_guides/prose/GUIDE.PROSE.AI_PATTERN_DETECTION.md`
- Create: `docs/superpowers/writing-kb/writing_mentor_kb/02_leaf_guides/prose/GUIDE.PROSE.VOICE_CONSISTENCY.md`

- [ ] **Step 1: Create prose subdirectory**

```bash
mkdir -p "docs/superpowers/writing-kb/writing_mentor_kb/02_leaf_guides/prose"
```

- [ ] **Step 2: Write GUIDE.PROSE.AI_PATTERN_DETECTION.md**

YAML frontmatter following KB leaf guide pattern:
```yaml
---
guide_id: GUIDE.PROSE.AI_PATTERN_DETECTION
title: AI Pattern Detection
doc_type: leaf_guide
topic: prose
---
```

Content: craft-oriented guide (not a detection checklist — that's in prose-lint-modules.md). Covers:
- Why AI patterns emerge in writing (statistical token prediction, safety training)
- The 6 pattern categories with brief explanation of WHY each is a tell
- How to distinguish genuine AI patterns from legitimate style (false positive awareness)
- When to use prose-editor for detailed analysis vs. self-editing with this guide

- [ ] **Step 3: Write GUIDE.PROSE.VOICE_CONSISTENCY.md**

YAML frontmatter:
```yaml
---
guide_id: GUIDE.PROSE.VOICE_CONSISTENCY
title: Voice Consistency
doc_type: leaf_guide
topic: prose
---
```

Content: craft-oriented guide covering:
- What voice IS (vs. style, tone, register)
- Common voice drift patterns (AI editing homogenization, revision smoothing, style guide over-correction)
- How to identify your own voice (the Voice Profile concept)
- When voice inconsistency is a problem vs. intentional modulation
- When to use prose-editor for voice analysis

- [ ] **Step 4: Commit**

```bash
git add docs/superpowers/writing-kb/writing_mentor_kb/02_leaf_guides/prose/
git commit -m "feat: add KB leaf guides for AI patterns and voice consistency"
```

### Task 7: Update ROUTER.REVISION.md

**Files:**
- Modify: `docs/superpowers/writing-kb/writing_mentor_kb/01_routing_cards/ROUTER.REVISION.md`

- [ ] **Step 1: Read current ROUTER.REVISION.md**

Read the full file to understand current routing entries and format.

- [ ] **Step 2: Add routing entries for new leaf guides**

Add to the Optional section (these are secondary loads, not primary revision tools):

```
- GUIDE.PROSE.AI_PATTERN_DETECTION (when symptom involves AI-sounding prose or generic voice)
- GUIDE.PROSE.VOICE_CONSISTENCY (when symptom involves lost voice, flat tone, or voice drift)
```

Follow existing format — no `.md` extension in routing entry names.

- [ ] **Step 3: Commit**

```bash
git add docs/superpowers/writing-kb/writing_mentor_kb/01_routing_cards/ROUTER.REVISION.md
git commit -m "feat: add prose leaf guide routing entries to ROUTER.REVISION"
```

### Task 8: Update specialist-routing-guide.md

**Files:**
- Modify: `~/.claude/skills/writing-guide/references/specialist-routing-guide.md`

- [ ] **Step 1: Read current specialist-routing-guide.md**

Read the full file.

- [ ] **Step 2: Update Section 1 (Decision Tree)**

Replace:
```
+-- Prose consistency / review need detected?
|     YES --> Offer prose-editor [FUTURE -- not yet available]
```

With:
```
+-- Prose consistency / review need detected?
|     YES --> Offer prose-editor
```

Add priority note: if both story-architect and prose-editor could match, story-architect takes priority.

- [ ] **Step 3: Update Section 3 (Entry Triggers)**

Replace the FUTURE placeholder with 4 entry triggers:
1. Explicit invocation — Writer pastes prose and requests review
2. Post-handoff invocation — story-architect handoff exists
3. Diagnostic escalation — Structural module surfaces prose-level problem
4. Session resume — Incomplete review artifact detected

- [ ] **Step 4: Add Section 5 negative heuristics for prose-editor**

Add: "prose-editor does not handle general craft questions about technique, genre selection, or story structure — those stay in writing-guide."

- [ ] **Step 5: Add Section 6 session state detection for prose-editor**

Add: "Check for incomplete `[nn]-[slug]-review.md` in `docs/writing/[project]/chapters/` directory at session start. Found: offer resume. Not found: no active review."

- [ ] **Step 6: Add detection heuristics**

Primary signal (keyword match): "review," "edit," "lint," "sounds like AI," "voice," "rhythm," "too wordy"
Secondary signal (text length): prose paste >50 words alongside review request
Tertiary signal: active handoff + draft question

- [ ] **Step 7: Commit**

```bash
git add ~/.claude/skills/writing-guide/references/specialist-routing-guide.md
git commit -m "feat: activate prose-editor in specialist routing guide"
```

### Task 9: Update escalation-triage.md

**Files:**
- Modify: `~/.claude/skills/writing-guide/references/escalation-triage.md`

- [ ] **Step 1: Read current escalation-triage.md**

Read the full file.

- [ ] **Step 2: Add structural-conflict type to escalation type table**

Add new row:
| `structural-conflict` | prose-editor | writing-guide, then writer decision | hold | Choose plan-revise or prose-revise |

Note: This type activates in Phase 2 only. Add a note: "Phase 2 activation — structural checks not available in Phase 1."

- [ ] **Step 3: Add Direction 3 payload schema reference**

Add a new section "Direction 3: prose-editor escalates to writing-guide" with the JSON payload schema from spec Section 4.9. Reference dispatch-and-escalation.md as the runtime reference.

- [ ] **Step 4: Add routing rule for structural-conflict**

Following the existing routing rules pattern, add: structural-conflict routes to writing-guide with plan-vs-prose decision template (from spec Section 5.3).

- [ ] **Step 5: Commit**

```bash
git add ~/.claude/skills/writing-guide/references/escalation-triage.md
git commit -m "feat: add Direction 3 escalation and structural-conflict type to triage"
```

### Task 10: Update artifact-schemas.md

**Files:**
- Modify: `~/.claude/skills/story-architect/references/artifact-schemas.md`

- [ ] **Step 1: Read current artifact-schemas.md**

Read the escalation payload schemas section.

- [ ] **Step 2: Add Direction 3 escalation payload schema**

Add as a new subsection after Direction 2:

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

- [ ] **Step 3: Commit**

```bash
git add ~/.claude/skills/story-architect/references/artifact-schemas.md
git commit -m "feat: add Direction 3 escalation payload schema to artifact-schemas"
```

---

## Chunk 4: Verification and Final Commit

### Task 11: Verify all files exist and are well-formed

- [ ] **Step 1: Verify prose-editor skill directory**

```bash
ls -la ~/.claude/skills/prose-editor/
ls -la ~/.claude/skills/prose-editor/references/
```

Expected:
```
SKILL.md
references/
  prose-lint-modules.md
  voice-profile-protocol.md
  review-protocol.md
  dispatch-and-escalation.md
```

- [ ] **Step 2: Verify line counts**

```bash
wc -l ~/.claude/skills/prose-editor/SKILL.md
wc -l ~/.claude/skills/prose-editor/references/*.md
```

Expected:
- SKILL.md: ~430 lines (under 500)
- prose-lint-modules.md: ~250 lines
- voice-profile-protocol.md: ~120 lines
- review-protocol.md: ~280 lines
- dispatch-and-escalation.md: ~170 lines

- [ ] **Step 3: Verify KB leaf guides**

```bash
ls -la docs/superpowers/writing-kb/writing_mentor_kb/02_leaf_guides/prose/
```

Expected: GUIDE.PROSE.AI_PATTERN_DETECTION.md, GUIDE.PROSE.VOICE_CONSISTENCY.md

- [ ] **Step 4: Verify frontmatter in SKILL.md**

Read first 15 lines of SKILL.md. Confirm:
- `name: prose-editor`
- `description:` is present and descriptive
- YAML block is properly delimited with `---`

- [ ] **Step 5: Verify routing update**

Read specialist-routing-guide.md. Confirm:
- No `[FUTURE]` text remains
- 4 entry triggers listed for prose-editor
- Detection heuristics present
- Negative heuristics present
- Session detection present

- [ ] **Step 6: Verify ROUTER.REVISION.md update**

Read ROUTER.REVISION.md. Confirm:
- GUIDE.PROSE.AI_PATTERN_DETECTION appears in routing entries
- GUIDE.PROSE.VOICE_CONSISTENCY appears in routing entries

- [ ] **Step 7: Verify escalation-triage.md update**

Read escalation-triage.md. Confirm:
- `structural-conflict` type present in escalation type table
- Direction 3 payload schema reference present
- Routing rule for structural-conflict present

- [ ] **Step 8: Verify artifact-schemas.md update**

Read artifact-schemas.md. Confirm:
- Direction 3 escalation payload schema present
- Schema matches spec Section 4.9

- [ ] **Step 9: Update workflow artifacts**

Update `docs/workflow/history.md` with task completion entry.

- [ ] **Step 10: Final commit**

```bash
git add docs/workflow/
git commit -m "chore: update workflow artifacts after prose-editor Phase 1 build"
```

---

## Execution Notes

- **Sequencing dependency:** This plan assumes story-architect reference files (artifact-schemas.md, etc.) exist. If they don't yet, prose-editor can still be built — its references to those files are consumption contracts, not hard dependencies for the skill file itself.
- **Testing:** Since these are Claude skill files (markdown), testing is done by invoking the skill in a conversation and verifying behavior. There are no unit tests — verification is behavioral.
- **Phase 2 preparation:** Phase 2 tasks (structural alignment, character consistency, full continuity ledger, voice deviation, handoff versioning) are NOT in this plan. They will be a separate plan after Phase 1 is validated in use.
