# Support log — one code review perch for darn and loom

## Interview

A darn run holds no interview: the darn writer reads the board entry and makes its own calls, recorded here and in the decision record.

## Rejected alternatives

- **Keep darn's perch keys in darn.yaml under the shared rows.** Rejected: the board entry asks for one key set, and darn already reads loom.yaml's judge and seed keys for the same perch.
- **Retire the darn perch keys without moving them.** Rejected: a task's or run profile's `run_settings` budget for darn would be refused or lost; the declared key move carries it.
- **Have the darn writer copy the board body into a new decision record section.** Rejected: it needs a discussion format change, and a writer-made copy can drift from the entry with nothing to catch it; a Go-written snapshot cannot.
- **Keep `lyx board get` as darn's answer key.** Rejected: the operator proposal asks that the perch read a file, as loom's perch does.
- **A new rubric marker carrying the recipe's answer key text.** Rejected: the rows' `artifact_paths` and `fasit` config already carry a per-recipe answer key, so a marker adds infrastructure for nothing.
- **Make a darn budget of 0 behave like loom's, a Bouncer settling at once.** Rejected: it adds a Describe session that rewrites a description no review changed.
- **Map the retired `Darn-*` row names onto the `Code-*` rows for resume.** Rejected: the run subdir changes from `darn` to `code`, so a resumed perch would find none of its round artifacts; a restart is the recipe's existing rule for retired rows.

## Review rounds

_No rounds yet._

## Question ledger

- **Which file holds the one key set?** Self-pick: loom.yaml's `code_*` keys, which darn's judge and seed already read. Resolved.
- **Where does the copied board scope live, apart from the writer's additions?** Self-pick: a Go-written `board-entry.md` snapshot in the discussion directory. Resolved.
- **Does darn's budget-0 shortcut survive?** Self-pick: yes, keyed on the `Code-Review` soft cap. Resolved.
- **How does `review_amend_max_bounces` move, given loom's is run-wide?** Self-pick: onto loom's run-wide key, under the default move rule that keeps a tuned loom value. Resolved.
