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
| **Premise Lock** | `02_leaf_guides/premise/GUIDE.PREMISE.VIABILITY.md`, `03_checklists/premise/CHK.PREMISE.VIABILITY.md` |
| **Character Commitment** | `02_leaf_guides/character/GUIDE.CHARACTER.WANT_NEED_MISBELIEF.md` |
| **Structure Approval** | `02_leaf_guides/structure/GUIDE.STRUCTURE.WEILAND_BEAT_MAP.md`, `02_leaf_guides/structure/GUIDE.STRUCTURE.TWO_HALVES_MAJOR_BEATS.md`, `02_leaf_guides/structure/GUIDE.STRUCTURE.MIDPOINT_SHIFT.md` |
| **Chapter Direction** | `03_checklists/scene/CHK.SCENE.PURPOSE.md` |

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
| **On topic** | Gate's KB files from gate-to-KB mapping (Section 2) | When writer discusses fields in that gate's domain |
| **On character work** | `characters/[name].md` for the character being discussed | When writer mentions a specific character by name |
| **On structure work** | `structure-map.md`, `cross-character-relationships.md` | When writer asks about structure |
| **On chapter work** | `chapters/[nn]-[slug].md` for the chapter being planned | When writer discusses a specific chapter |
| **On handoff** | All artifact paths (index only, not full contents) | When generating handoff-context.md |
| **Never during planning** | `handoff-context.md`, `chapters/[nn]-[slug]-review.md` | These are write-only or prose-editor-facing artifacts |

---

## 6. KB Coverage Gaps

| Genre | Gate Coverage | Notes |
|---|---|---|
| **narrative-fiction** | Full coverage across all 4 gates | KB was built primarily for fiction |
| **memoir** | Partial -- Character Commitment gate thin | Only narrative-fiction has full KB coverage. Memoir uses general resolution criteria. |
| **creative-nonfiction** | Partial -- KB guides are fiction-oriented | No dedicated CNF guides exist. |
| **long-form-essay** | Minimal -- most KB backing absent | Essay-specific concepts have no dedicated KB guides. |

