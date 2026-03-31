# Genre Vocabulary — Substitution Tables

Reference for adapting surface language by genre tag. Loaded when friction questions fire or gate prompts render.

---

## 1. Genre Tag Definitions

- **`narrative-fiction`** — Novels, short stories, novellas; invented characters and events.
- **`memoir`** — First-person retrospective nonfiction organized around a wound or reckoning.
- **`creative-nonfiction`** — Reported or researched nonfiction with literary structure (long-form journalism, immersion, essay-as-investigation).
- **`long-form-essay`** — Argument-driven or inquiry-driven nonfiction organized around a question, not a narrative arc.

---

## 2. Substitution Table

At render time, replace the generic term (left column) with the genre-specific term from the writer's selected genre tag.

| Generic Term    | narrative-fiction   | memoir                    | creative-nonfiction       | long-form-essay                    |
|-----------------|---------------------|---------------------------|---------------------------|------------------------------------|
| protagonist     | protagonist         | narrator / I              | subject / central figure  | author's position                  |
| want            | goal                | driving question          | investigative lens        | central argument                   |
| wound           | backstory wound     | formative experience      | entry point               | animating tension                  |
| arc             | character arc       | arc of understanding      | arc of inquiry            | movement of thought                |
| inciting event  | inciting incident   | rupture or trigger        | precipitating event       | the problem that won't resolve     |
| stakes          | story stakes        | personal stakes           | stakes of the record      | stakes of the claim                |
| theme           | theme               | meaning made              | frame of significance     | thesis territory                   |

---

## 3. Gate Question Language Adaptation Rules

- Friction trigger conditions are identical across all genres.
- Only the surface nouns change via the substitution table above.
- At render time, replace the generic term with the genre-specific term from the writer's selected genre tag.
- If no genre tag is set, fall back to the generic term.

### Example

Generic gate question:

> "What force is working against what your **protagonist** wants most?"

Rendered for `memoir`:

> "What force is working against what your **narrator** wants most?"

Rendered for `creative-nonfiction`:

> "What force is working against what your **subject** wants most?"

Rendered for `long-form-essay`:

> "What force is working against what your **author's position** wants most?"

---

## 4. Genre-Normative Structural Differences

This is the most important conceptual note in this reference. Different genres resolve the "misbelief-equivalent" differently. Gate questions for the misbelief-equivalent must account for these differences.

### Fiction
Misbelief is **overcome**. The protagonist sheds their false belief through the story's events.

### Memoir
The Lie the Self Told is **exposed**. The narrator gains distance from the survival story they held.

### Creative Nonfiction
The Reporter's Complicity is **interrogated**. The writer's assumptions are confronted by the material.

### Long-Form Essay
The Governing Contradiction may be **held permanently**. This is genre-normative, NOT a design gap. The essay form allows paradox to remain unresolved. Gate questions for the misbelief-equivalent must account for this — do not force resolution where the genre does not require it.

---

## 5. Hybrid Genre Handling

- Writer selects one primary genre tag at session intake.
- Primary tag governs ALL gate requirements, character frame vocabulary, and friction question language.
- Secondary genre characteristics can be noted in `story-manifest.md` as a freeform field.
- At any gate, the writer can request KB resources from a secondary genre tag — writer-initiated override only.
- Hybrid genre handling beyond this is out of scope for v1.
