# Card 10 — batten-create-precondition-mapping

**Edit:**
- `internal/battencli#createRefusal` -> `func createRefusal(err error, slug, taskBranch string) error`
- `internal/battencli#battenCLI.wire`
- `internal/battencli#TestCreateRefusal_LeftoverBranchRemedyNeverNamesCheckout`

**Uses:**
- `plan:internal/battenshed#PreconditionError`
- `internal/fabricengine#ErrSourceDirty`
- `internal/fabricengine#ErrBranchExists`
- `internal/fabricengine#ErrRemoteLeftover`

**Intent:**
Map the add's typed refusals to Worktree-Create's precondition halt (D11), so a dirty source met after the start check, or a task branch left after a kept pair was removed by hand, hands back with its way forward and never reaches a tackler.
`createRefusal` takes the slug and the task branch name (`cfg.BranchPrefix + slug`, which `battenCLI.wire`'s create closure already holds) and returns a `*battenshed.PreconditionError` wrapping the cause for:

- an `*fabricengine.ErrSourceDirty`: unmet `the source worktree has uncommitted tracked changes: <paths>`, way forward `commit or revert them in the source worktree, then run "lyx batten run <slug>" again` (PATTERN-no-denied-recovery: it names no `git stash`);
- an `*fabricengine.ErrBranchExists`: unmet `task branch <branch> is left from an earlier pair of this slug`, way forward `run "lyx batten cancel <slug> --remove" from the prime, which archives and deletes it, then "lyx batten run <slug>"`;
- an `*fabricengine.ErrRemoteLeftover` whose `Branch` equals the task branch: the same as the line above, naming the branch on origin.

Every other error, a remote leftover of the records side included, passes through unchanged, as today.
The old leftover-branch rewording, which named `git branch -D` and `git push origin --delete`, is replaced by the cancel; the grep for its text found only this card's test asserting it.
`battenCLI.wire`'s create closure passes the slug and the task branch.
`TestCreateRefusal_LeftoverBranchRemedyNeverNamesCheckout` keeps its never-`lyx fabric checkout` assertion and its pass-through and nil rows, and its leftover rows now assert a `*battenshed.PreconditionError` naming `lyx batten cancel some-slug --remove`; new rows cover a bare and a wrapped `*ErrSourceDirty` naming its paths and `commit or revert`, an `*ErrRemoteLeftover` on the task branch mapped, and one on another branch passed through.

**ImpactSummary:** `createRefusal` has one production caller, the create closure in `battenCLI.wire`, and one test; the halt reaches the driver only through card 9's producer branch.
