# lym beam merge-parent files a task that merges a beam's parent into it

Bringing a beam's parent branch into the beam used to mean writing a merge task by hand.
`lym beam merge-parent <beam>` now files that task with one call, at any time, and two default reed buttons in the beam group reach it and `lym beam land`.

## What changes

- **`lym beam merge-parent <slug>`** (`internal/beamcli/mergeparent.go`) files a darn task in the beam's project.
  Its run merges the beam's parent branch, the `ParentBranch` of the beam pair's origin record, into the task branch, and its landing carries the merge into the beam.
  The beam's dobby starts it as any member.
  - It runs anywhere in the hub through the same prime resolution and entry check as `lym beam land`.
    It refuses a task, a grouping project, a done beam and a beam with no pair here, as `land` does.
    It also refuses a beam whose origin record names no parent branch, naming `lym fabric reconcile`, and a beam that holds an open merge task, naming that task's `lym batten status` and `lym batten cancel --remove`.
  - The task's slug is the beam's slug with `-merge-in`, numbered from `-2` when the slug is taken, and the beam part is cut so the slug fits `boardengine.MaxSlugLength`.
    The task carries the beam's labels, so the board's type-label rule holds without a hard-coded label.
  - The board gives a new task the draft status, so the filing releases the task with a second write, as `fronttask` does.
  - The task's `run_settings` turn the Code-Review perch on.
    `loom.code_max_bounces` is set to the hub's resolved budget, at least one round, and `loom.code_hard_max_bounces` stays above it.
    `run_settings` layer over any profile, so a profile that turns darn's perch off cannot reach this run.
  - The body, which the decision record carries as the goal of both the writer and the perch, has the writer run `lym fabric merge-in <parent>`, settle the conflicts and make the gate pass, and change nothing else.
    It also says the perch reviews the merge alone and that checking the project's goal is no part of the task.
- **The Code-Review rubric** (`contracts/stencils/loom/loom-rubric-code-review.md`) narrows the review range for a merge task.
  The reviewer reads each merge commit through `git show --remerge-diff`, plus every commit after the merge, and never the parent's own commits.
  Without this, the range `review_base..HEAD` would put every parent commit since the beam forked under review.
- **Reed's default `beam` button group** gains `merge-in`, running `lym beam merge-parent {slug}`, and `land`, running `lym beam land {slug}`, each behind a confirm prompt, in both platform templates.
  The default `prime` group already lists the beam group's other buttons without these two, so the prime does not fall back to the beam group, and it stays as it is.
- The package doc of `internal/beamcli`, the verb's cobra help and `docs/overview.md`'s beam module line describe the verb.

pawl is unchanged: it neither starts a merge task nor waits for one, and its Parent-Align brings the parent in at the landing either way.

## Design decisions

- **The darn writer makes the merge, not a landing row's merge fixer.**
  The goal asks for conflicts and gate failures to be settled "with the merge fixer".
  The merge fixer belongs to Parent-Align and Finalize, and those rows merge the task's own parent, the beam, under that parent's lock, and record the merge in a record Finalize reads.
  Pointing a landing row at the beam's parent would take a second parent branch, lock and merge record through `landingshed` and the darn recipe.
  Instead, the task is a plain darn run: its writer runs `lym fabric merge-in` and settles the conflicts, and its `Darn-Write` verify gate's cold gate fixer clears the gate failures.
  Drift runs on the task as on any darn run whenever `drift.enabled` holds.
- **The buttons use `{slug}`, not `{beam}`.**
  The goal writes `lym beam merge-parent {beam}`, but reed fills `{beam}` with the beam's anchor directory.
  In a beam session `{slug}` is the beam's worktree name, which is its board slug.
- **A merge task is found by its project and slug suffix.**
  The board has no field that marks one.
  The open-task refusal reads the task's `project` and its slug's `-merge-in` suffix, which `merge-parent` alone writes.

## Tests

`internal/beamcli/mergeparent_test.go` drives the sequence over the land test's fake hub, which now also carries a board, the filed tasks and the hub's Code-Review caps.
It covers each refusal, the perch turned on from an off budget and kept from the hub's, the next number after a done merge task, the slug cut for a long beam slug, and the beam's labels carried over.
`internal/reedengine/config_test.go` pins the two new default beam buttons in both templates.
