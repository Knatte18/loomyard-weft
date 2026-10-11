# Darn: rename the writer row to Darn-Write and close the perch gaps

The darn recipe named both itself and its writer row `Darn`.
The writer row is now `Darn-Write`, matching loom's `-Write` rows and the row's engine, `DarnWrite`.
This change also closes the gaps the `Darn-Review` perch left: a stale pull request description, a hardcoded verify budget, the default perch budget, and an untested spent budget.

## Row rename, and resume across it

- `internal/loomshed`: `NameDarn` becomes `NameDarnWrite` (`"Darn-Write"`), with its interrupt policy.
- `contracts/recipes/darn-recipe.yaml`: the row, `Preflight`'s `on_done` and `PR-Gate`'s `on_stuck` use the new name.
- The writer's friction note is now `Darn-Write.md`, and every way forward names the new row: `lyx loom goto --to Darn-Write`, the resume reason, the `lyx loom reject` help and the refusal spec.
- An in-flight darn run whose status file still names `Darn` stays resumable.
  `shedengine.RenameProducers` is a new generic status-file migration.
  It rewrites retired row names in `current_producer`, `pause_before`, `pause_after` and each history entry's producer, under the run lock.
  It leaves a file that names no retired row untouched, and takes no lock for it.
  `loomrecipe.DarnRetiredRows` maps `Darn` to `Darn-Write`.
  `loomcli`'s `armAt` applies it to every darn run before routing, lightweight verbs included, so `status` and `goto` see the new name too.
  A live driver holding the run lock is left alone with a warning, because it runs the binary that wrote the old name.

## Describe after the perch

`Darn-Burler` runs `fix-scope: source` and can change the code after the writer described it.
`Darn-Bouncer`'s `on_done` now goes to a `Describe` row, which goes on to Publish, as loom's `Webster-Bouncer` does.

- The row reuses the `Describe` engine, its description gate and its commit, under landing.yaml's `describe` model.
- `wireDarn` replaces the loom `DescribeSpec` with `landingshed.DarnDescribeSpec`, because a darn run has no decision record or webster run record.
  The new stencil, `landing-template-describe-darn`, reads the board entry and the diff.
  It also reads the perch's run directory: when a round's circling decision is an `accept`, that round's findings become items to check by hand.
- `landingshed.DescribeSpec` and `DarnDescribeSpec` share one composer.
- With `review_max_bounces: 0`, `Darn-Write` still routes straight to Publish, so the writer's own description stands.
- A `PR-Gate` rejection bounces to `Darn-Write`, so a rework passes the perch and `Describe` again before Publish.
  Publish already refreshes an open pull request's title and body when they differ from the description, in darn and loom alike; `TestPublish_OpenPullRequest` pins it.
  `loom-final-summary-spec.md` said Publish reads the description only when no pull request exists; it now states the refresh.

## Budgets

- `Darn-Burler`'s verify gate no longer hardcodes `attempts: 3`.
  `NewDarn` stamps `darn.yaml`'s `verify_attempts` onto the verify gates of both `Darn-Write` and `Darn-Burler`.
- `review_max_bounces` defaults to `1` in `internal/darnengine/template.yaml`, the darnengine docs, `docs/overview.md` and the test env.
  An existing hub's `darn.yaml` that sets the key keeps its value.

## Spent perch budget

A new test, `TestEscalation_DarnPerchBudgetSpentEscalatesThenAcceptReachesDescribe`, drives a darn run past the default budget of 1 with no decision record on disk.
The Bouncer escalates to the parent (cause `budget`) with a rendered brief and parent notice, and writes no decision record.
An `accept` then settles the perch and runs `Describe`.

The engine path held up, but the parent's instructions on that path did not fit a darn run:

- The escalation brief told the parent fork to record each design call with `lyx loom decision add`, which always refuses in a darn run.
  The brief now records them only when the decision record exists, and otherwise has the fork name them in its report.
- `lyx loom decision add`'s no-record refusal now says a darn run never has a record, and the refusal spec row matches.

## To check by hand

- A darn run parked at the old `Darn` row: after the binary update, run `lyx loom status` in its worktree, and check that it reports `Darn-Write` and that `lyx loom resume` continues it.
- One darn run end to end with a non-zero perch budget, to see the `Describe` session's description on the pull request.
