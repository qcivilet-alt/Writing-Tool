# Escalation Format

All inter-skill escalations use this format:

  Escalation: [trigger] in [location].
  [Description of what was found and why it matters.]
  See [artifact path] for full findings.
  Writer decision needed: [specific question].

## Trigger Types

| Trigger | Routes To | Severity |
|---|---|---|
| structural-gap | story-architect at relevant gate | flag |
| voice-drift | writing-guide for craft guidance | advisory |
| continuity-contradiction | writer chooses plan-revise or prose-revise | hold |
| diagnostic | writing-guide diagnostic mode | hold |
| ghostwriting-risk | enforced locally, never routed | non-bypassable |

## Routing Rules

Structural rework returns to source. prose-editor does not fix structural
problems at the prose layer -- it routes back to story-architect.

When multiple escalations trigger in the same pass, holds take priority.
Advisory findings are bundled into the review artifact, not emitted as
separate escalation payloads.
