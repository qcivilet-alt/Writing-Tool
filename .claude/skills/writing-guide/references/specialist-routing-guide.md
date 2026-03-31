# Specialist Routing Guide

Runtime reference for writing-guide: when to offer a specialist skill and when to handle a query directly.

---

## 1. Routing Decision Tree

```
Writer query arrives at writing-guide
  |
  +-- Structural planning need detected?
  |     YES --> Offer story-architect
  |
  +-- Prose consistency / review need detected?
  |     YES --> Offer prose-editor
  |     (Priority: if both story-architect and prose-editor match, story-architect takes priority)
  |
  +-- Neither?
        --> Handle in writing-guide (no specialist needed)
```

## 2. Entry Triggers for story-architect

1. **Explicit invocation** -- Writer asks to start a new project, develop an existing idea, or work through a structural problem. writing-guide offers story-architect; writer confirms.
2. **Toll-gate escalation** -- writing-guide reaches a toll gate and the writer's project has no viable premise on record. writing-guide surfaces the gap and offers to invoke story-architect.
3. **Session resume** -- Writer returns to an in-progress story-architect session. writing-guide detects the open session artifact (`story-architect-session.json`) and asks if the writer wants to continue from the last completed gate.

## 3. Entry Triggers for prose-editor

1. **Explicit invocation** -- Writer pastes prose and requests review. writing-guide detects prose-review need and offers prose-editor. Writer confirms.
2. **Post-handoff invocation** -- story-architect generates `handoff-context.json`. writing-guide surfaces handoff availability; writer confirms.
3. **Diagnostic escalation** -- writing-guide or story-architect surfaces a prose-level problem (voice drift, rhythm issue) that exceeds craft guidance scope.
4. **Session resume** -- writing-guide detects incomplete review artifact (`[nn]-[slug]-review.md` with `last_completed_pass` < total passes) and offers to continue from last completed pass.

**Detection heuristics:**
- Primary signal (keyword match): "review," "edit," "lint," "sounds like AI," "voice," "rhythm," "too wordy"
- Secondary signal (text length): prose paste >50 words alongside review request
- Tertiary signal: active handoff + draft question

## 4. Offer Framing

- Transitions are **offered, not forced** -- the writer can decline any invocation.
- The writer always starts at writing-guide (single entry surface).
- Specialist invocations are writer-triggered, not automatic.

Example offer:

> "It sounds like you're working on story structure. I can help with general craft questions here, or if you'd like a structured planning session, I can invoke story-architect. What would you prefer?"

## 5. What writing-guide Does NOT Do

- Does not attempt to provide story-structure guidance that story-architect handles (premise development, act structure, scene sequencing, character-arc mapping).
- Does not attempt prose review that prose-editor handles (line edits, voice consistency, pacing diagnostics).
- This boundary exists because diluting specialist depth into the hub layer produces worse outcomes than routing.

**prose-editor negative heuristics:** prose-editor does not handle general craft questions about technique, genre selection, or story structure — those stay in writing-guide. If the writer asks "how do I write better dialogue?" without submitting prose for review, that's a craft question for writing-guide, not a prose-editor invocation.

## 6. Session State Detection

**story-architect:**
- Check for `story-architect-session.json` in the project directory at session start.
- **Found**: An active story-architect project exists -- surface it and offer resume.
- **Not found**: No active project -- handle general craft queries normally.

**prose-editor:**
- Check for incomplete `[nn]-[slug]-review.md` in `docs/writing/[project]/chapters/` directory at session start.
- **Found**: An active prose-editor review exists -- surface it and offer resume from last completed pass.
- **Not found**: No active review -- handle normally.
