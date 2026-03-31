# KB Gate Map

How story-architect loads Knowledge Base files per gate, what fields each gate requires per genre, gap severity rules, and context budget management.

---

## 1. Loading Rule

- KB files load when a gate becomes active, not before.
- KB files are unloaded when the gate closes.
- Story-architect uses KB files **internally** to evaluate gate readiness. It does **not** surface KB content as craft education to the writer.
- If the writer asks a craft question, story-architect routes to `writing-guide` instead.

---

## 2. Gate-to-KB Mapping

All KB paths are relative to: `docs/superpowers/writing-kb/writing_mentor_kb/`

| Gate | KB Files Loaded |
|---|---|
| **Premise Lock** | `02_leaf_guides/GUIDE.PREMISE.VIABILITY.md`, `03_checklists/CHK.PREMISE.VIABILITY.md` |
| **Character Commitment** | `02_leaf_guides/GUIDE.CHARACTER.WANT_NEED_MISBELIEF.md`, `02_leaf_guides/GUIDE.CHARACTER.AGENCY_AND_DECISION_PRESSURE.md` |
| **Structure Approval** | `02_leaf_guides/GUIDE.STRUCTURE.WEILAND_BEAT_MAP.md`, `02_leaf_guides/GUIDE.STRUCTURE.TWO_HALVES_MAJOR_BEATS.md`, `02_leaf_guides/GUIDE.STRUCTURE.MIDPOINT_SHIFT.md` |
| **Chapter Direction** | `03_checklists/CHK.SCENE.PURPOSE.md`, `05_dependency_maps/MAP.CONTINUITY.LEDGER_TEMPLATE.md` |

---

## 3. Required Field Sets Per Gate x Genre

Each cell lists the fields that **must** be resolved before the gate can pass for that genre.

### Premise Lock

| Genre | Required Fields |
|---|---|
| **narrative-fiction** | `core_tension`, `protagonist_stake`, `thematic_question`, `inciting_premise` |
| **memoir** | `core_tension`, `protagonist_stake`, `thematic_question`, `narrative_voice` |
| **creative-nonfiction** | `core_tension`, `thematic_question`, `narrative_voice` |
| **long-form-essay** | `core_tension`, `thematic_question`, `narrative_voice`, `argument_claim` |

### Character Commitment

| Genre | Required Fields |
|---|---|
| **narrative-fiction** | `protagonist_desire`, `protagonist_wound`, `antagonist_function`, `character_arc_type` |
| **memoir** | `protagonist_desire`, `protagonist_wound`, `narrator_stance`, `key_relationships_mapped` |
| **creative-nonfiction** | `protagonist_desire`, `protagonist_wound`, `narrator_stance` |
| **long-form-essay** | `narrator_stance`, `speaker_ethos` |

### Structure Approval

| Genre | Required Fields |
|---|---|
| **narrative-fiction** | `act_structure`, `inciting_incident_placed`, `midpoint_reversal`, `climax_sketch` |
| **memoir** | `act_structure`, `climax_sketch`, `time_structure` |
| **creative-nonfiction** | `act_structure`, `section_logic`, `argument_arc` |
| **long-form-essay** | `act_structure`, `chapter_count_range`, `section_logic`, `argument_arc`, `tonal_consistency_check` |

### Chapter Direction

| Genre | Required Fields |
|---|---|
| **narrative-fiction** | `scene_objective`, `point_of_view_confirmed`, `entry_exit_beats`, `tension_raised`, `character_state_delta` |
| **memoir** | `scene_objective`, `point_of_view_confirmed`, `character_state_delta`, `sensory_anchor` |
| **creative-nonfiction** | `chapter_argument`, `point_of_view_confirmed`, `factual_research_flag`, `transitional_logic` |
| **long-form-essay** | `chapter_argument`, `point_of_view_confirmed`, `factual_research_flag`, `transitional_logic` |

---

## 4. Gap Severity Assignments

Severity is **static** per gate x field -- not computed dynamically. Writers know in advance which gaps will be critical.

| Severity | Definition | Examples | Behavior in Rigorous | Behavior in Workshop |
|---|---|---|---|---|
| **Critical** | Absence makes gate's downstream outputs logically undefined | `core_tension`, `act_structure`, `point_of_view_confirmed` | Gate hold, no override | Gate hold; deferral requires `reason_type: structural-experiment` + coherence review |
| **Structural** | Creates dependency risk -- downstream work possible but expensive to fix later | `character_arc_type`, `time_structure`, `argument_arc` | Gate hold | Gate hold with deferral path |
| **Advisory** | Quality signal, not a structural dependency | `cast_necessity_check`, `tonal_consistency_check` | Never a gate hold | Never a gate hold -- listed as craft notes |

---

## 5. Context Budget Management

### Tiered Loading Strategy

| Tier | What Loads | When |
|---|---|---|
| **Always** | `story-manifest.md` | Every session, every turn -- project identity and gate state |
| **On gate entry** | Gate's required-field reference from `artifact-schemas.md`, KB files from gate-to-KB mapping (Section 2) | When that gate becomes active |
| **On character work** | `characters/[name].md` for the character being discussed | When writer mentions a specific character by name |
| **On structure work** | `structure-map.md`, `cross-character-relationships.md` | When Structure Approval gate is active or writer asks about structure |
| **On chapter work** | `chapters/[nn]-[slug].md` for the chapter being planned | When Chapter Direction gate is active for that chapter |
| **On handoff** | All artifact paths (index only, not full contents) | When generating `handoff-context.json` |
| **Never during planning** | `handoff-context.json`, `chapters/[nn]-[slug]-review.md` | These are write-only or prose-editor-facing artifacts |

### 60% Capacity Warning

If the current session context exceeds **60% capacity** with loaded artifacts, the skill surfaces a warning and asks the writer which artifacts to keep vs. unload. It does **not** silently drop files.

---

## 6. KB Coverage Gaps

| Genre | Gate Coverage | Notes |
|---|---|---|
| **narrative-fiction** | Full coverage across all 4 gates | KB was built primarily for fiction |
| **memoir** | Partial -- Character Commitment gate thin | `GUIDE.CRAFT.MEMOIRIST_DISTANCE.md` is a **STUB** (not yet written). The "Earned Distance" field uses general resolution criteria as workaround. |
| **creative-nonfiction** | Partial -- KB guides are fiction-oriented | No dedicated CNF guides exist. Skill evaluates using analogous fiction concepts + genre vocabulary substitution. |
| **long-form-essay** | Minimal -- most KB backing absent | Essay-specific concepts (Animating Question, Persona, Governing Contradiction) have no dedicated KB guides. Skill relies on `character-frames.md` definitions + general resolution criteria. |

---

## 7. Token Budget Estimates Per Gate

Estimated token load per gate (KB files + relevant reference file sections + always-loaded story-manifest):

| Gate | KB Files (~tokens) | Reference Sections (~tokens) | story-manifest (~tokens) | Total Estimate |
|---|---|---|---|---|
| **Premise Lock** | ~2,500 (2 files) | ~800 (artifact-schemas premise fields + friction premise triggers) | ~500 | **~3,800** |
| **Character Commitment** | ~3,000 (2 files) | ~1,200 (character-frames + friction character triggers) | ~500 | **~4,700** |
| **Structure Approval** | ~4,500 (3 files) | ~1,000 (friction structure triggers + structure-map schema) | ~500 | **~6,000** |
| **Chapter Direction** | ~2,000 (2 files) | ~800 (chapter schema + friction triggers subset) | ~500 | **~3,300** |

These are rough estimates. The 60% capacity threshold is based on the model's available context window. With a 200K context window, 60% = ~120K tokens -- well above any single gate load. The risk is **cumulative**: multiple gates' worth of artifacts staying loaded across a long session.
