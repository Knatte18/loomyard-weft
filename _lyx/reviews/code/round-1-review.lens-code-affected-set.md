```yaml
lens: code-affected-set
outcome: reported
```

Task: lens code-affected-set, the set of code, tests and docs forced by each change.

**Per change, the set found**

- `docs/overview.md` gate bullet (the focus item): covered. It says a `--card` scope under the `go` scoper vets each other tier tag that adds a file to a card package, and that a failing tier stops no later tier, vet or guard run. `doc.go`'s Vet step section, `scope.go` `tagTiers`/`tagAddsFile`, `tiers.go` `planTierRuns`/`vetStepRun` and `run.go` do exactly this.
- `docs/overview.md`, gate-bullet clause "exits with the first failing run's code": covered. `run.go:135-150` records the first failing run's code, and `runLabel` gives `vet tierN`.
- Parent merge: covered. `git diff 5e36064..main --stat -- internal/gatecli` is empty, so the parent changed nothing under `internal/gatecli`, and the parent's `overview.md` edits are PATTERN link retargets that leave the gate bullet alone.
- New `tierRun.vetStep` field: covered. Every reader of `guardStep` was checked.
  - `run.go:140` dispatch: a vet run goes to `runListed`, as the design says.
  - `run.go:201` `runLabel`: updated for `vet`.
  - `run.go:303` `tierNumbers`: updated to leave out vet runs.
  - `run.go:80` and `run.go:95` read `sc.guardStep`, not the run field, so they are unaffected.
  - `tierrun.go:52` and `run.go:394` set `RunRecord.GuardStep` from `guardStep` only. A vet record is told apart by `ScopeForm == "vet"`, which matches the decision record.
  - `green.go:347` `recordPass` switches on `guardStep` only. A vet run takes the default branch and records its packages through `expandPackages` under the vet command's own fingerprint. This is consistent with the design.
- New `scopeVet` constant and the `scope.tagTiers` field: covered. Only `computeScope`'s card branch sets it, and `planTierRuns` reads it only under `form == scopeCard` and the `go` scoper. `doc.go` says the same.
- `doc.go` verb paragraph and guard-step line ("after the last listed tier and vet run"): updated.
- `planTierRuns` doc comment, `runLabel` and `tierNumbers` comments: updated.
- Tests: `tiers_test.go`, `scopes_integration_test.go` and `test_integration_test.go` are touched by the diff. I did not map them to behaviours, which is the tests lens's job.
- Stencils, fixtures and other docs: no reader of `ScopeForm` values or of the closing-line text outside `internal/gatecli` turned up in what I searched. I made no wider search of docs or stencils for `scopeGuard` or `guard tierN` strings.

**Findings:** none.

**Tool calls**
1. `grep` for `guardStep|scopeGuard|vetStep|scopeVet` across `internal` non-test files: to find every reader of the changed run field, which my context lacked outside `gatecli/scope.go`, `tiers.go` and `run.go`.
2. `sed -n` on `green.go:335-365` and `run.go:130-150`: `recordPass`'s switch and the run loop's dispatch were not in my context.
