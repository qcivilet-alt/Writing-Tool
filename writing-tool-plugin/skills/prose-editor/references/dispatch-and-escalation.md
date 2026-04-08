# Dispatch and Escalation

Concern-to-pass routing, sub-module selection, escalation payload schemas, and handoff loading protocol for the prose-editor skill.

---

## 1. Dispatch Map

### Concern-to-Pass Dispatch Table

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

## 2. Escalation

See `../../shared/escalation-schemas.md` for the unified escalation format.

When multiple escalations trigger in the same pass, holds take priority. Advisory findings are bundled into the review artifact.

prose-editor does NOT modify planning artifacts or resolve conflicts — it presents them for writer decision.

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
