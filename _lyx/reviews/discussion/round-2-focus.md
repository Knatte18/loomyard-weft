---
round: 2
focus:
  - "Pre-walk loading of moved keys: `configengine.Layers` runs `ValidateLocal` on every local file, which refuses a path absent from the template. Settle whether D4 or D2 states how shuttle's loader treats `claude_prompt_cache_ttl` and `claude_prompt_cache_ttl_roles` in a `shuttle.local.yaml` before reconcile has moved them, whether `lyx shuttle stop` and strand removal still work in that pair, and what bounds any admitted or refused key."
  - "Sparse-layer collisions in D4: settle whether a target key equal to the template default is overwritten only at base layers, what happens at a worktree or other sparse layer where that value may be an explicit override, and whether every overwrite is reported."
  - "D8's guarantee that a forgotten `reedengine.New` site is a compile error: settle whether it holds for a nil or empty scrub set, and whether the Decision states what guards that case."
---

Focus targets the design-class gap and the two decision-level changes; text-only corrections need no extra attention.
