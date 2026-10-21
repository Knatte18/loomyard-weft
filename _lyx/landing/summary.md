# Shipped stencils name no LoomYard PATTERN entry or lyx CLI rule

Every hub receives the shipped stencils and specs, but several named LoomYard's own `pattern/PATTERN-*.md` files and a `lyx` CLI rule.
In a hub without those files, such as quarry, the plan reviewer went searching the filesystem for `PATTERN-cli-cobra.md` and `PATTERN-test-economy.md`.
The shipped text now names no PATTERN entry, `pattern/` file or CLI-shape rule.
LoomYard's own reviews keep those rules, because its `PATTERN.md` reaches reviewers through the PATTERN directive.

Closes #795

## Changes

- `loom-rubric-plan-review.md`: the CLI-shape bullet is replaced by a PATTERN-fit bullet.
  When the repository has a `PATTERN.md`, each card is checked against its entries and the `pattern/` background files the plan names as relevant, and a finding names the entry.
  In a repository without a `PATTERN.md` the check does not apply.
- `loom-rubric-darn-review.md`, `loom-rubric-webster-review.md`, `loom-rubric-merge-review.md` and `loom-rubric-plan-review.md`: the test-economy bullet drops its `PATTERN-test-economy` citation.
- `loom-template-plan.md`, `loom-plan-spec.md` and `burler-step-fix.md`: the same citation is dropped from the card coverage rule and the fix-adds-a-test rule.
- `shed-template-driver-guide.md`: the stranded-branch repair no longer cites `PATTERN-fabric-git`; it says the raw `git` calls are allowed repairs and the driver makes no commits.

## Design decisions

- The test-economy rules keep their listed findings and lose only the citation.
  A redundant or missing test is a finding in any repository, and LoomYard's stricter parts stay in `pattern/PATTERN-test-economy.md`.
- The plan rubric does not choose which background file applies.
  The plan agent already makes that choice from the hub's `PATTERN.md`, and the rubric checks against what the plan names.
- No marker is added or removed, so each stencil's marker set is unchanged.

## Not covered

- The `fabric-vocabulary-name` gate in `loom-plan-spec.md` describes a Go finding that names `PATTERN-fabric-vocabulary` in every hub.
  Making it repo-neutral changes Go code and is left to a separate task.
- Nothing mechanical stops a later stencil edit from citing a LoomYard PATTERN entry again; review is the guard.

## To check

`lyx gate test ./contracts/...` was not run as part of this description.
