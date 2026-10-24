## Interview

A darn run holds no interview; the writer read the board entry and the profile code and made its own calls.

## Rejected alternatives

- Per-recipe key names, such as `darn_code_max_bounces`: rejected by the operator's 2026-10-24 decision for one block per recipe.
- A `default_profile` key in the loom and darn config modules: a task's `run_settings` could set it, where it is meaningless, and Seed-Child would read it from the hub's layers before the profile is known.
- A reconcile migration from the old profile shape: the seed-only contract never rewrites a present file, and the old profiles do not map onto the ladder; the board entry asks for the hub's profiles to be rewritten, not migrated.
- A board upsert check that the profile covers the task's recipe: the board resolves no recipe, by its own design, so Seed-Child alone refuses it.
- An entry gate to turn a perch off at 0 rounds: the perch keys already settle approved at 0.

## Review rounds

_No rounds yet._

## Question ledger

- Does a round count of 0 turn a perch off through profile settings alone? Resolved: yes, the `<perch>_max_bounces` keys document that 0 runs no review round.
- How do the Code and Darn columns coexist under one `code_*` key? Resolved by the operator: one block per recipe.
- Where does the per-recipe default profile live? Self-pick: a reserved `defaults` entry in `profiles.yaml`.
- What does a speed perch at 0 rounds take as its hard cap? Self-pick: 1, rounds plus one, since a hard cap of 0 means "take the review default".
- Who rewrites the hub's `profiles.yaml`? Self-pick: the parent, after the deploy, since a task session works only in its own worktree.
