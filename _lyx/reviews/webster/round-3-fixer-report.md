# Webster-Burler round 3 — fixer report

## Fixed

### F1 — the recorded-parent-branch reader duplicates the `wiring.go` closure and no test reaches it (`00f9a5850`)

The webster `RunDeps.ParentBranch` closure in `internal/loomcli/wiring.go` now calls `readRecordedParentBranch(location)`, so one function owns the origin-record read and its absent-record wording.

One part of the finding's premise is off: the `wiring.go` closure did have a test, `TestWire_WebsterParentBranchReadsPairOrigin` in `wiring_parentbranch_integration_test.go`, covering the recorded-branch path over a real records repository.
With the closure delegating, that test now reaches the real `readRecordedParentBranch`.
I added the absent-record case to that existing test as one assertion, made before the record is written, which pins the exact error text.
No new top-level test was added, so the test budget is unchanged.

Verified: `lyx gate test ./internal/loomcli` and `lyx gate test --tags integration ./internal/loomcli` green; `gofmt -l` and `lyx loom lint-comments` clean.

## Disputed

None.

## Deferred

None.
