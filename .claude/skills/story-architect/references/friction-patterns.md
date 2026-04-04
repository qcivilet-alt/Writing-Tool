# Friction Patterns

The behavioral engine that drives story-architect's friction questions. SKILL.md invokes this file by reference whenever friction questions are active.

Dependencies:
- Field names from `artifact-schemas.md`
- Genre substitution from `genre-vocabulary.md`

---

## 1. Gap Scan

Before each friction question, scan the required fields for the writer's project:

Priority: contradictions > empty fields > underdeveloped fields
Within each level: Premise > Character > Structure > Theme
Recency: skip fields addressed in the last 2 turns
Fire exactly one question per turn.

A field is "empty" if it contains a placeholder (TBD, "not sure yet",
"something about", "maybe") or a single word where a sentence is needed.

A field is "underdeveloped" if it lacks a complete sentence with specific
details -- proper nouns, concrete objects, or named relationships.

A contradiction exists when two populated fields conflict: want vs. obstacle,
stakes vs. ending, theme vs. arc, tone vs. structure.

---

## 3. Friction Trigger Tables

All questions use generic vocabulary. Genre substitution happens at render time via `genre-vocabulary.md`.

### Premise Triggers

| Condition | Question |
|---|---|
| `core_tension` empty or placeholder | "What force or situation is actively working against what your [protagonist] wants most?" |
| `core_tension` has no opposing agent | "Who or what specifically stands between your [protagonist] and what they're after — and why can't they simply step aside?" |
| `stakes` empty | "If your [protagonist] fails completely, what is permanently lost or broken?" |
| `stakes` present but reversible with no cost | "What in your story makes that loss irreversible?" |
| `inciting_event` empty or lacks specific trigger | "What specific event forces your [protagonist] into this story rather than letting them stay in their ordinary life?" |

### Character Triggers

| Condition | Question |
|---|---|
| `want` empty or abstract noun only | "What does your [protagonist] actively want at the start of this story — stated as a specific goal?" |
| `need` empty | "What does your [protagonist] actually need that they don't yet know they need?" |
| `want` and `need` identical | "If want and need are the same thing, what creates the internal tension?" |
| `wound/backstory_driver` empty | "What happened to your [protagonist] before this story begins that makes them exactly wrong — or exactly right — for this situation?" |
| `change_vector` empty | "Where does your [protagonist] end up internally that they couldn't have reached at the beginning?" |

### Structure Triggers

| Condition | Question |
|---|---|
| `opening_condition` empty | "What is the specific state of your [protagonist]'s world at the moment this story begins?" |
| `ending_type` empty | "Do you know yet how this story ends — not plot mechanics, but the emotional or thematic resolution?" |
| `midpoint_shift` empty | "What happens in the middle that reframes what the story is actually about — not just escalates it?" |
| `ending_type` contradicts `stakes` | "Your ending doesn't seem to cost your [protagonist] anything — what does the resolution actually demand from them?" |

### Theme Triggers

| Condition | Question |
|---|---|
| `central_question` empty | "What question does this story hold open — not answer, but genuinely sit with — by the last page?" |
| `central_question` is a statement | "Can you reframe this as a question the story refuses to answer cleanly?" |
| `traceable_event` empty | "Where in the actual events of your story does this theme get tested most severely?" |
| `central_question` conflicts with `change_vector` | "Your theme asks one question but your [protagonist]'s arc answers a different one — which is this story actually about?" |

---

## 4. Dormant Rule

Five conditions when no friction question fires:

1. **Intake still in progress.** The minimum viable brief has not yet been established. Friction questions require a baseline to scan against.

2. **Writer just answered a friction question.** The question was answered in the prior turn and the engine is still in cooldown. Let the writer breathe.

3. **Writer asked the skill a direct question.** Answer it. Do not redirect with friction.

4. **Writer declared a generative sprint.** ("I'm going to brainstorm for a bit.") The skill enters listener mode until the sprint ends. See Section 7.

5. **Brief is sufficiently developed.** All required fields are populated above clause and specificity thresholds with no contradictions. There is nothing to probe.

---

## 5. Answer Validation

When a writer's answer fails minimum-viable resolution, the response follows this exact pattern:

```
[What was heard] — restates the writer's answer in one phrase
[What's missing] — names the specific gap the answer didn't close
[Re-framed probe] — asks the question from a different angle
```

Example:

```
You said: "She wants revenge."
That's a feeling, not a goal — it doesn't tell us what she's trying to make happen
in the world. What specific action is she trying to take, and what would success
look like from the outside?
```

The skill does NOT generate an example of a resolved answer — that would be ghostwriting. It can describe the structure of a resolved answer ("a specific action, a named target, a concrete consequence") without providing one.

### Re-ask Limit

After three failed attempts at the same field, the skill names the pattern directly:

"We've circled this question three times. It may not be answerable yet — do you want to park it and come back, or explore what's blocking you?"

---

## 6. Hollow Compliance Detection

If a writer is answering every friction question in under ten words with no self-generated specificity, the skill names the pattern after three consecutive thin answers — not accusatorially, but observationally:

"Your last few answers are technically complete but very brief. Would it help to take one of these deeper before we move on?"

This is advisory only. The skill does not gate on hollow compliance. It surfaces the observation and moves on.

---

## 7. Sprint Mode

### Entry Trigger

Writer says any variation of:

- "I want to think out loud"
- "Let me brainstorm for a bit"
- "I'm going to explore this for a minute"
- "Sprint"

### Behavior During Sprint

The skill enters listener mode. It reflects back key phrases, tracks new story material for later gap scanning, and does not challenge or question. It does not log gap flags during a sprint — the material is exploratory, not commitments.

### Exit Trigger

Writer says any variation of:

- "Done"
- "Okay, what did you notice?"
- "Let's check what we have"
- Any explicit return signal

### Post-Sprint

The skill summarizes what emerged from the sprint, identifies any new fields that were populated or implied, and resumes the friction engine from the current gap scan.

---

## 8. Gate Holds

A gap becomes a gate hold when:
1. The field is required for the gate being cleared
2. The session mode is Rigorous or Workshop
3. The gap is a contradiction or an empty critical field

Exploratory mode: no gate holds. Gaps annotate as [UNRESOLVED: reason].
Workshop mode: holds can be deferred with a one-sentence reason.

---

## 9. Gap Severity

See `kb-gate-map.md#gap-severity-assignments` for the canonical severity table (Critical / Structural / Advisory).

---

## 10. Multi-Gap Answer Processing

When a writer's response addresses multiple open fields simultaneously, story-architect processes all of them — not just the one it asked about.

After receiving a writer response, the skill runs the gap scan on all currently open fields against the new content. Any field that passes minimum-viable resolution is updated and its annotation removed.

The skill acknowledges what was resolved before firing the next friction question:

```
That resolved three things at once:
  checkmark core_tension — updated
  checkmark protagonist_stake — updated
  checkmark inciting_premise — updated

One remaining gap before Premise Lock is ready:
  thematic_question — [next friction question fires here]
```

The multi-gap scan ensures the writer is never re-asked about something they already addressed, even if they addressed it in an answer to a different question.
