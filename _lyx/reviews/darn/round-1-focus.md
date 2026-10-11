---
round: 1
focus:
  - "`shedengine.RenameProducers` and its `armAt` call in `loomcli`: does the migration rewrite every place a row name is stored (`current_producer`, `pause_before`, `pause_after`, history producers), leave a file with no retired row untouched without taking the lock, and behave correctly when a live driver holds the run lock?"
  - "Row rename completeness: is any `Darn` row-name reference left in the diff's reach (recipe edges, `NameDarnWrite` users, friction note path, `goto`/resume/`reject` help, refusal spec, docs, stencils, tests), and is the interrupt policy carried over to `NameDarnWrite`?"
  - "The `Describe` row after `Darn-Bouncer`: does `wireDarn` wire it with `DarnDescribeSpec`, does `review_max_bounces: 0` still route `Darn-Write` straight to Publish, and does a `PR-Gate` rejection bounce to `Darn-Write` as the board entry asks?"
  - "`landing-template-describe-darn` and the shared composer: does the stencil read the perch run directory correctly for `accept` decisions, and did the shared composer change loom's `DescribeSpec` output?"
  - "Verify budget and default: does `NewDarn` stamp `verify_attempts` on both gates with no hardcoded `attempts: 3` left, and is the `review_max_bounces` default of 1 stated consistently in the template, darnengine docs, `docs/overview.md`, recipe comments and test env while an existing `darn.yaml` value is kept?"
  - "The escalation brief and `lyx loom decision add` refusal change: is the no-record behavior correct for darn without altering loom's brief, and does the refusal spec row match the code?"
  - "Summary claims against the diff, including the Publish refresh of an open pull request in darn and loom (`TestPublish_OpenPullRequest`) and the `loom-final-summary-spec.md` change; and whether the board entry's request to check loom's rejection path is satisfied."
  - "Test economy: do the new tests duplicate existing coverage (name the covering test), and is any added behavior uncovered, e.g. the `RenameProducers` no-op and locked-run paths, `armAt` on lightweight verbs, `DarnRetiredRows`?"
  - "Repository rules: docs in the same commit (`doc.go`, `docs/overview.md`, `PATTERN.md` if a new invariant), the test budget file `cmd/lyx/testdata/test-budget.yaml` if a package passes its row, and semantic line breaks in changed `.md` files."
---

Seed focus from the board entry and the change description; the base diff review still applies in full.
