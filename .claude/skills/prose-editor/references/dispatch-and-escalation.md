# Dispatch and Escalation

Concern-to-pass routing, sub-module selection, escalation payload schemas, and handoff loading protocol for the prose-editor skill.

---

## 1. Dispatch Map

### Concern-to-Pass-to-Sub-Module Routing

| Concern Tag | Passes Activated | Pass 5 Sub-Modules Activated |
|---|---|---|
| `sounds-too-ai` | Pass 5, Pass 3 (Voice Coherence) | AI-ism Detection (full), Diction Audit (compact), Sentence Variation (compact) |
| `needs-tightening` | Pass 5 | Diction Audit (full), Cliche Scanner (compact), Echo Detection (compact) |
| `rhythm-off` | Pass 5 | Sentence Variation (full), Echo Detection (compact), Readability/Pacing (if fiction/long-form) |
| `full-review` | All five passes (1, 2, 3, 4, 5) | All six sub-modules (full reference versions) |
| `quick-polish` | Pass 5 only | All six sub-modules (compact versions), capped at top 5 findings |

### Sub-Module Selection Rules

- Load only sub-modules from `prose-lint-modules.md` matching the dispatch concern tag.
- **Full** = complete sub-module with all checklist items and thresholds.
- **Compact** = abbreviated version: top 3-5 highest-signal checks only.
- Unused sub-modules are not processed, preserving context budget.

### Genre Calibration

| Genre | Tolerance | Priority | Special Rules |
|---|---|---|---|
| Fiction | Fragments, unconventional punctuation, stylistic passive OK | Show-don't-tell checks active; pacing analysis active | Don't flag intentional rule-breaking |
| Business | Low tolerance for passive, cliches, wordiness | Clarity and concision priority | Skip show-don't-tell |
| Essay/opinion | Preserve strong voice and personality | Flag hedging aggressively -- opinion needs conviction | Longer sentences OK if earned |
| Marketing | Very low tolerance for cliches | Specificity and concrete claims priority | Short paragraphs expected |
| Academic | Passive voice and hedging OK where epistemically appropriate | Clarity and precision over personality | Don't penalize complexity for expert audience |

### Audience Calibration (Readability Targets)

| Audience | Target Grade Level |
|---|---|
| Adolescent / young adult | 6-8 |
| General public | 10-12 |
| Professional peers | 10-12 |
| Academic reviewers | 12-16 |
| No audience specified | 10-12 (default) |

### Calibration Override Rule

Genre and audience override raw readability scores. Do not tell an academic their prose is "too complex" when their audience expects complexity. Readability metrics serve the writer's context, not the other way around.

---

## 2. Escalation Payload Schemas

Direction 3: prose-editor to writing-guide.

### Trigger Types

| Trigger | Condition | Severity | Routes To |
|---|---|---|---|
| `voice-drift` | Sustained voice departure across 3+ passages | Advisory | writing-guide (craft guidance) |
| `structural-conflict` | Prose contradicts locked gate field | Hold | writing-guide -> writer decision (plan-revise or prose-revise) |
| `continuity-contradiction` | Prose contradicts established ledger fact | Hold | writing-guide -> writer decision |
| `diagnostic` | Premise-level problem detected in prose | Hold | writing-guide (Diagnostic mode) |

### JSON Payload Schema

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

### Concurrent Escalation Rule

When multiple escalations are triggered in the same review pass, holds take priority. Hold-severity escalations are emitted as payloads. Advisory-severity findings are bundled into the review artifact but not emitted as separate escalation payloads.

### Scope Boundaries

prose-editor does NOT:

- Modify planning artifacts.
- Route directly to story-architect (always goes through writing-guide).
- Resolve conflicts -- presents them for writer decision.

---

## 3. Handoff Loading Protocol

### Post-Handoff Entry Flow

**Step 1.** Locate latest `handoff-context-v[N].json` in `docs/writing/[project]/handoff/`. Latest = highest version number N by filename sort.

**Step 2.** Validate `schema_version` MAJOR on load. MAJOR mismatch triggers an error surfaced to the writer, not silent degradation. Offer context-only mode as fallback.

**Step 3.** Extract from the handoff payload:
- `project.genre_tag`
- `gates` (status of each)
- `artifacts` (paths)
- `session_mode`
- `open_questions`

**Step 4.** Load locked gate fields as hard constraints. Do not flag prose that complies with locked decisions.

**Step 5.** Load artifact paths. Read on demand per pass, not all at once.

**Step 6.** Surface `open_questions` with `priority: high` to the writer before starting review.

### Handoff Version Determination

Highest N by filename sort of `handoff-context-v[N].json` files in the handoff directory. No timestamp parsing -- pure lexicographic sort on the version number extracted from the filename.

### Schema Version Validation

Compare MAJOR version only. Minor version differences are accepted without warning. Major version mismatch triggers an error with a context-only fallback offer. Context-only mode loads genre, gates, and artifact paths but skips constraint enforcement from gates.

### Edge Cases

**All gates `not_started` or `skipped`:** Proceed with an advisory note that the planning foundation is thin. Treat all findings as advisory or flag severity, never hold. The writer chose to skip planning -- respect that decision while noting the limitation.

**No text provided:** Soft rejection. Prompt the writer to supply text before review can begin. Do not attempt analysis on empty input.

**No handoff file found:** Operate without planning context. All passes run in standalone mode. Escalation payloads that reference gate fields are suppressed since there are no gates to conflict with.
