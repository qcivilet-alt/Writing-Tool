# Review Protocol

Five-pass review logic and post-edit verification reference for the prose-editor skill.

---

## Table of Contents

1. [Five-Pass Review Sequence](#1-five-pass-review-sequence)
2. [Per-Pass Execution Protocol](#2-per-pass-execution-protocol)
3. [Review Artifact Write Protocol](#3-review-artifact-write-protocol)
4. [Post-Edit Verification Protocol](#4-post-edit-verification-protocol)
5. [Continuity Check Protocol (Phase 1 -- Read-Only)](#5-continuity-check-protocol-phase-1--read-only)

---

## 1. Five-Pass Review Sequence

### Pass Execution Order

| Pass | Name                  | What It Checks                                                       | Phase              |
|------|-----------------------|----------------------------------------------------------------------|--------------------|
| 1    | Structural Alignment  | Chapter events/tensions vs. chapter plan and locked gates            | Phase 2            |
| 2    | Character Consistency | Character behavior vs. committed character frames                    | Phase 2            |
| 3    | Voice Coherence       | Prose voice vs. voice profile + character voice anchors              | Phase 1            |
| 4    | Continuity Check      | Prose facts vs. continuity ledger; flag candidate new facts          | Phase 1 (read-only)|
| 5    | Prose Quality         | AI-isms, diction, rhythm, cliches, echoes, readability/pacing       | Phase 1            |

### Override Rules

- Dispatch can reorder passes based on concern tag. Example: `sounds-too-ai` runs Pass 5 first.
- Dispatch can skip passes entirely. Example: `quick-polish` runs only Pass 5 in compact mode.
- Passes 1-2 are automatically skipped when no planning artifacts are available.
- If Pass 5 finds 10+ hold-severity issues, pause and present findings before running further passes.

### Per-Pass Checkpoint

After each pass completes, briefly report finding count by severity. If findings are minimal (<3 total), note this and consider skipping lower-priority passes.

### Phase 1 Skip Declarations for Passes 1-2

When Passes 1 or 2 are reached and planning artifacts are unavailable, emit the corresponding declaration and advance to the next pass:

- **Pass 1:** "Pass 1 skipped: structural alignment checks require Phase 2 implementation."
- **Pass 2:** "Pass 2 skipped: character consistency checks require Phase 2 implementation."

---

## 2. Per-Pass Execution Protocol

### Common Structure

Every pass follows the same execution frame:

1. **Load** -- Read the required reference files and artifacts into context.
2. **Scan** -- Walk the submitted prose, applying the pass-specific checks.
3. **Record** -- Write each finding in the standard output format (see below).
4. **Calibrate** -- Adjust severity using the genre table and audience readability targets.
5. **Report** -- Emit the per-pass checkpoint summary.

### Finding Output Format

Each finding is a structured record:

```
[Pass N] <severity> | <location> | <finding summary>
  Detail: <explanation of the issue>
  Suggestion: <concrete revision direction, never a rewrite>
```

Location uses the format `P<paragraph>S<sentence>` (e.g., `P3S2` = paragraph 3, sentence 2). For span-level issues, append word range: `P3S2:W4-7`.

### Severity Levels

| Severity | Label   | Meaning                                                                 |
|----------|---------|-------------------------------------------------------------------------|
| H        | Hold    | Blocks further work. Must be addressed before subsequent passes.        |
| F        | Flag    | Significant issue. Should be addressed but does not block.              |
| A        | Advisory| Worth noting. Writer may accept or dismiss at their discretion.         |
| O        | Observe | Pattern noted for awareness. No action expected.                        |
| E        | Echo    | Recurring pattern across multiple locations. Attached to another level. |

Echo (E) is a modifier, not a standalone severity. It attaches to another severity level to indicate the issue recurs. Example: `[A][E]` means an advisory-level issue that appears in multiple places.

### Severity Assignment Rules

- Default to Advisory unless the issue materially changes meaning (Flag) or introduces a factual contradiction (Hold).
- Promote to Flag when the issue would confuse or mislead a target-audience reader.
- Promote to Hold only for factual errors, continuity breaks, or structural misalignment with locked planning gates.
- Attach Echo when the same pattern appears 3+ times in the submitted text.

---

### Genre Calibration Table

Genre context adjusts what counts as an issue and at what severity.

| Genre          | Tolerance                                          | Priority                              | Special Rules                            |
|----------------|----------------------------------------------------|---------------------------------------|------------------------------------------|
| Fiction        | Fragments, unconventional punctuation, stylistic passive OK | Show-don't-tell checks active; pacing active | Do not flag intentional rule-breaking    |
| Business       | Low tolerance for passive, cliches, wordiness      | Clarity and concision                 | Skip show-don't-tell                     |
| Essay/opinion  | Preserve strong voice and personality               | Flag hedging aggressively             | Longer sentences OK if earned            |
| Marketing      | Very low tolerance for cliches                      | Specificity and concrete claims       | Short paragraphs expected                |
| Academic       | Passive voice and hedging OK where epistemically appropriate | Clarity and precision over personality | Do not penalize complexity for expert audience |

When genre is unspecified, default to Fiction tolerance with show-don't-tell checks inactive.

### Audience Readability Targets

| Audience                  | Target Grade Level |
|---------------------------|--------------------|
| Adolescent / young adult  | 6-8                |
| General public            | 10-12              |
| Professional peers        | 10-12              |
| Academic reviewers        | 12-16              |
| No audience specified     | 10-12 (default)    |

Grade level is advisory context for severity calibration, not a mechanical gate. Prose that reads above the target range is not automatically flagged -- flag only when elevated complexity serves no clear purpose.

---

### Pass 1: Structural Alignment

- **What it checks:** Whether chapter events, scene beats, and narrative tensions in the prose match the chapter plan and locked planning gates.
- **What it loads:** Chapter plan artifact, locked gate definitions, story-architect handoff context (if present).
- **Calibration:** Fiction and creative nonfiction only. Skip for genres without narrative structure.
- **Phase 2 dependency:** This pass requires planning artifacts. If unavailable, emit skip declaration.

### Pass 2: Character Consistency

- **What it checks:** Whether character behavior, speech patterns, knowledge, and emotional state are consistent with committed character frames.
- **What it loads:** Character frame artifacts, character voice anchors, scene context from chapter plan.
- **Calibration:** Fiction and memoir. Severity escalates for named characters with locked frames.
- **Phase 2 dependency:** This pass requires character frame artifacts. If unavailable, emit skip declaration.

### Pass 3: Voice Coherence

- **What it checks:** Whether the prose voice matches the writer's voice profile and any character voice anchors for POV characters.
- **What it loads:** Voice profile, character voice anchors (for fiction with POV characters).
- **Calibration:** All genres. Voice drift from the profile is Flag severity by default. Intentional voice shifts within a piece (e.g., tonal modulation) are noted as Observe.

### Pass 4: Continuity Check

- **What it checks:** Whether facts stated or implied in the prose contradict the continuity ledger. Identifies candidate new facts for ledger addition.
- **What it loads:** continuity-ledger.md (read-only). See Section 5 for full protocol.
- **Calibration:** Fiction and memoir primarily. For nonfiction, factual consistency checks apply to claims and data references.

### Pass 5: Prose Quality

- **What it checks:** AI-isms, diction problems, rhythm flatness, cliche density, word/phrase echoes, readability relative to audience, pacing issues.
- **What it loads:** Voice profile (for baseline comparison), genre and audience from intake context.
- **Calibration:** All genres. Genre table governs tolerance thresholds.

#### Pass 5 Preamble: Five Diagnostic Questions

After dispatch selects sub-modules but before sub-module execution, run these five diagnostic questions against the submitted text:

1. **Is anyone home?** -- Can you detect a specific person making specific choices, or could this have been written by anyone about anything?
2. **Can you see anything?** -- Is the language anchored in visible, physical reality, or does it float in abstraction?
3. **Does it vary?** -- Do sentence lengths, structures, and rhythms change, or does the prose hum at one frequency?
4. **Does it take a position?** -- Does the writer commit to a stance, or present permanent hedged neutrality?
5. **Does anything surprise?** -- Is there a single phrase, image, or structural choice you did not see coming?

**Meta-diagnostic rule:** If all five answers are negative, record a Flag-severity meta-finding: "Passage registers as uniformly weak across all quality dimensions. Consider `full-review` concern tag for comprehensive analysis."

<!-- Source: Paper B S7, "The five diagnostic questions" -->

---

## 3. Review Artifact Write Protocol

### When to Write

Review artifacts are written to disk after each pass completes. This serves two purposes:

1. **Checkpoint/recovery** -- If the session is interrupted, completed pass results are preserved.
2. **Accumulated context** -- Later passes can reference earlier findings without re-scanning.

### Write Notice / Write Receipt Protocol

Every disk write follows a two-step confirmation sequence:

1. **Write Notice** (before writing):
   - State the target file path.
   - State what will be written (pass number, finding count, artifact type).
   - Example: "Write Notice: Writing Pass 3 results (7 findings) to `review-artifact-[timestamp].md`."

2. **Write Receipt** (after writing):
   - Confirm the file path written.
   - Confirm byte count or line count as verification.
   - Example: "Write Receipt: `review-artifact-[timestamp].md` updated. Pass 3 block: 42 lines written."

### Review Artifact Header

The review artifact header is auto-populated from intake context. Required fields:

```
---
review_id: <auto-generated>
source_file: <path or identifier of submitted text>
genre: <genre from intake>
audience: <audience from intake>
concern_tags: <tags from dispatch>
voice_profile: <path to voice profile, or "none">
timestamp: <ISO 8601>
---
```

If any intake field is missing, record it as `"not specified"` rather than omitting the field.

### Artifact Structure

The review artifact accumulates pass results sequentially:

```
[Header block]

## Pass N: <Pass Name>
Status: completed | skipped
Finding count: <count by severity>
Findings:
  [finding records]

## Pass N+1: ...
```

Each pass appends its block. Completed passes are never overwritten by later passes.

---

## 4. Post-Edit Verification Protocol

When a writer returns with revised text after receiving review findings, run the post-edit verification sequence. This protocol is adapted from Toolkit Module 9.

### Verification Checks

Run the following checks in order against the revised text, comparing it to the original submission:

1. **Voice drift check** -- Compare revised text against the voice profile. Has the revision moved closer to or further from the writer's established voice?

2. **Meaning drift check** -- Compare the intent of revised passages against the original. Has the meaning shifted in ways the writer likely did not intend?

3. **Over-smoothing check** -- Identify productive quirks, intentional roughness, or distinctive patterns from the original that have been stripped in revision. Not all rough edges are flaws.

4. **Homogenization check** -- Has sentence-level variation (length, structure, rhythm) been flattened? Revision should preserve or improve variation, not eliminate it.

5. **Inserted AI-isms check** -- Scan the revised text for newly introduced AI-characteristic patterns that were not present in the original. Common indicators: generic intensifiers, hedge-then-assert patterns, list-of-three constructions, abstract emotional labeling.

6. **Iteration assessment** -- Produce a summary:
   - Issues resolved (reference original finding IDs).
   - New issues introduced (record as new findings).
   - Net severity improvement or regression.

### Iteration Limit

Maximum 3 revision-verification cycles per passage. After the third cycle, emit an over-editing warning:

"Third revision cycle reached. Continued iteration risks diminishing returns. Recommend the writer step away from this passage and return with fresh perspective, or accept the current state and move forward."

### Over-Correction Checks

These checks detect false solutions -- revisions that simulate quality without achieving it. All are Advisory severity.

1. **Forced quirkiness** -- Performative personality layered on top rather than emerging from within. The prose equivalent of a toupee: the effort to appear natural is itself what appears unnatural. `[A]`

2. **Gratuitous sensory detail** -- Showing without narrative function. Description must serve a purpose (mood, characterization, foreshadowing, worldbuilding), not merely exist to demonstrate showing-not-telling. `[A][E]`

3. **Deliberately choppy prose** -- Overcorrection toward uniformly short sentences in response to rhythm feedback. Rhythm requires variation between long and short, not replacement of one monotony with another. `[A]`

4. **Purple prose as proof of humanity** -- Ornamental language deployed specifically to distinguish the text from AI output. Ornament without purpose is ornament without value regardless of motive. `[A][E]`

5. **Deliberately inserted errors** -- Typos, grammatical mistakes, or awkward constructions introduced to appear human. Performing imperfection is itself a form of inauthenticity. `[O][E]`

### Diagnostic Principle

Every false solution attempts to simulate the artifacts of authenticity without the substance underneath. The solution is never cosmetic -- it is having something specific to say and the craft to say it precisely.

<!-- Source: Paper B S9, "False solutions and the overcorrection trap" -->

---

## 5. Continuity Check Protocol (Phase 1 -- Read-Only)

### Skip Condition

If no `continuity-ledger.md` exists in the project context (context-only mode with no prior review history), skip Pass 4 with the following declaration:

"Pass 4 skipped: no continuity ledger found. Continuity checking requires an established ledger from prior review sessions."

### When Ledger Exists

1. **Load** -- Read `continuity-ledger.md` into context. Do not modify it.
2. **Scan** -- Compare facts stated or implied in the submitted prose against ledger entries.
3. **Flag contradictions** -- Any prose fact that contradicts a ledger entry is recorded as Hold severity. Include both the prose reference and the ledger entry in the finding record.
4. **Identify candidate new facts** -- Facts in the prose that are not present in the ledger but could be ledger-worthy are collected in a Continuity Candidates section of the review artifact.

### Continuity Candidates Format

```
## Continuity Candidates (Pass 4)

| # | Fact                          | Location | Ledger Category | Confidence |
|---|-------------------------------|----------|-----------------|------------|
| 1 | <stated or implied fact>      | P2S4     | <suggested>     | high/med   |
| 2 | ...                           | ...      | ...             | ...        |
```

Confidence reflects how explicitly the fact is stated:
- **High** -- Directly and unambiguously stated.
- **Medium** -- Implied or inferable but not directly stated.

Low-confidence inferences are not recorded as candidates.

### Phase 1 Write Guard

The prose-editor skill operates in read-only mode with respect to the continuity ledger during Phase 1. The skill must not write to `continuity-ledger.md` under any circumstances.

If the execution path would result in a ledger write:

1. **Halt** the write operation.
2. **Redirect** the candidate entries to the Continuity Candidates section of the review artifact.
3. **Log** the redirect: "Write guard activated: ledger write redirected to review artifact Continuity Candidates section."

Ledger updates are the responsibility of a separate ledger-management process (Phase 2 scope). The prose-editor's role is strictly to identify candidates and flag contradictions.
