---

## Pipeline Controls

[lane_a]
superpowers_planning: none
require_verification_gate: false
require_code_review_gate: false
step_checkpoint_prompt: false
require_worktree: false
dry_run: false

[lane_b]
superpowers_planning: writing-plans
require_verification_gate: true
require_code_review_gate: true
review_frequency: pre_merge
step_checkpoint_prompt: true
auto_proceed_intake: false
require_worktree: true
dry_run: false

[lane_c]
superpowers_planning: brainstorming+writing-plans
require_verification_gate: true
require_code_review_gate: true
review_frequency: per_step
step_checkpoint_prompt: true
auto_proceed_intake: false
require_worktree: true
dry_run: false
coherence_review_required: true
