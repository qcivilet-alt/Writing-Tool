# Dispatch and Escalation

Concern-to-pass routing, sub-module selection, escalation payload schemas, and handoff loading protocol for the prose-editor skill.

---

## 1. Dispatch Map

### Concern-to-Pass-to-Sub-Module Routing

| Concern Tag | Passes Activated | Pass 3 Sub-Modules Activated |
|---|---|---|
| `sounds-too-ai` | Pass 3, Pass 1 (Voice Coherence) | AI-ism Detection (full), Diction Audit (compact), Sentence Variation (compact) |
| `needs-tightening` | Pass 3 | Diction Audit (full), Cliche Scanner (compact), Echo Detection (compact) |
| `rhythm-off` | Pass 3 | Sentence Variation (full), Echo Detection (compact), Readability/Pacing (if fiction/long-form) |
| `full-review` | All three passes | All six sub-modules (full reference versions) |
| `quick-polish` | Pass 3 only | All six sub-modules (compact versions), capped at top 5 findings |

### Sub-Module Selection Rules

- Load only lint modules from `references/lint/` matching the dispatch concern tag.
- **Full** = complete sub-module with all checklist items and thresholds.
- **Compact** = abbreviated version: top 3-5 highest-signal checks only.
- Unused sub-modules are not processed, preserving context budget.

### Genre Calibration
See `review-protocol.md#genre-calibration-table` for genre tolerance, priority, and special rules.

### Audience Calibration
See `review-protocol.md#audience-readability-targets` for grade-level targets by audience.

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
1. Locate `handoff-context.md` in `docs/writing/[project]/`
2. Read YAML frontmatter for gates, artifacts, open_questions
3. Load cleared gate fields as hard constraints
4. Load artifact paths. Read on demand per pass.
5. Surface open_questions to the writer before starting review.

### Edge Cases

**All gates `not_started`:** Proceed with an advisory note that the planning foundation is thin. Treat all findings as advisory or flag severity, never hold.

**No text provided:** Prompt the writer to supply text before review can begin.

**No handoff file found:** Operate without planning context. All passes run in standalone mode.
