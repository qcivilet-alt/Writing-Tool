# Escalation Triage

How writing-guide interprets and routes escalation payloads from specialist modules.

## 1. Escalation Type Table

| Type | Source | Routes to | Severity | Writer action required |
|---|---|---|---|---|
| `structural-gap` | any | story-architect | flag | Resolve missing field at appropriate gate |
| `voice-drift` | prose-editor | writing-guide | advisory | Review KB guidance; revise if writer agrees |
| `continuity-contradiction` | prose-editor | writing-guide -> story-architect | hold | Writer chooses plan-revise or prose-revise path |
| `gate-hold` | story-architect | story-architect (held in place) | hold | Provide required field inputs to pass gate |
| `diagnostic` | story-architect / writing-guide | writing-guide (Diagnostic mode) | hold | Premise-level structural contradiction requiring foundational rework |
| `structural-conflict` | prose-editor | writing-guide, then writer decision | hold | Choose plan-revise or prose-revise (Phase 2 activation — structural checks not available in Phase 1) |
| `ghostwriting-risk` | any | enforced locally, not routed | non-bypassable | N/A -- skill refuses the request in place |

**On ghostwriting-risk:** Each module enforces its own anti-ghostwriting constraint locally. A `ghostwriting-risk` escalation is NOT routed to writing-guide for central adjudication -- it is handled at the point of detection. No escalation pathway exists because routing implies the possibility of override; there is none.

## 2. Payload Interpretation Rules

- Escalation payloads arrive as JSON (schema in `story-architect/references/artifact-schemas.md#escalation-payload-schemas`)
- The `trigger` field determines escalation type
- The `context` object carries enough information for writing-guide to frame the issue for the writer
- writing-guide does NOT attempt to resolve escalations internally -- it triages and routes

## 2.5 Direction 3: prose-editor escalates to writing-guide

Escalation payloads from prose-editor follow the Direction 3 schema defined in `story-architect/references/artifact-schemas.md#direction-3`. The payload carries:

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

Runtime reference for payload construction: `prose-editor/references/dispatch-and-escalation.md`.

---

## 3. Routing Rules

**Structural rework returns to source.** If prose-editor identifies a structural problem, it does not attempt to resolve it at the prose layer. It routes back to story-architect at the relevant gate. Surface-level fixes at the prose layer mask foundational contradictions -- the problem returns in a later chapter.

Routing logic by type:

- `structural-gap`: Route to story-architect. The gap exists in plan artifacts; only story-architect gates can enforce resolution.
- `voice-drift`: Retain in writing-guide. Present KB guidance to the writer; revise only if the writer agrees the drift is unintentional.
- `continuity-contradiction`: Present decision to writer (see Section 4). Route depends on writer choice.
- `gate-hold`: No routing needed. The hold is already in story-architect; the writer provides inputs there.
- `diagnostic`: Activate Diagnostic mode in writing-guide. Foundational rework may require returning to early story-architect gates.
- `structural-conflict`: Present plan-vs-prose decision to writer (same template as continuity-contradiction in Section 4). Route depends on writer choice: plan-revise goes to story-architect at relevant gate; prose-revise returns to prose-editor. Phase 2 activation — not available in Phase 1.
- `ghostwriting-risk`: No routing. Enforced locally at detection point.

## 4. Writer-Facing Communication -- Plan-vs-Prose Decision

When a continuity-contradiction escalation arrives, writing-guide presents the writer with a concrete decision:

```
Structural conflict detected:
prose-editor found that [character] acts in chapter [N] in a way that
contradicts the committed arc in character-profiles.md.

You have three paths:
  1. Revise the plan -- return to story-architect at Character Commitment
     gate and update the arc
  2. Revise the prose -- the plan is right; the chapter drifted. Return
     to the chapter with the committed arc in view
  3. Defer -- flag this conflict and continue drafting; resolve before
     the next prose-editor review
```

The writer chooses. If "Revise the plan," writing-guide invokes story-architect at the relevant gate with the escalation payload as context.

## 5. Return Loops -- Non-Linear Re-Entry

When the writer revises a plan artifact after an escalation, the pipeline does not restart from scratch. It re-enters at the relevant gate:

1. Writer re-enters story-architect at the relevant gate
2. story-architect runs gate assessment after writer updates the artifact
3. story-architect generates `handoff-context-v[N+1].json`
4. On next prose-editor invocation, it compares loaded context version against latest -- flags mismatch, asks writer to confirm
5. Scoped second pass resumes against confirmed version

This ensures that plan changes propagate forward without requiring a full pipeline restart, while still giving the writer explicit control over version alignment.
