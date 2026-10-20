# Fixer-report, darn round 1

## Changes

- F1: `buildCommand` in `internal/impactset/derive.go` no longer emits the untagged `go test` step, so the set runs only `go test -tags integration` after the vet steps.
  Its comment, the command description in `internal/impactset/doc.go` and the `gateCommand` helper in `internal/impactset/derive_test.go` follow.
  Fixed as proposed.
  Commit: `impactset: fix F1`.
- F2: fixed as proposed.
  The refresh moved into a new `refreshStored` in `internal/gatecli/prebuild.go`, which re-stats the entry after a successful `os.Chtimes` and reports a not-exist result as not refreshed, so `copyIntoPlace` republishes it.
  Commit: `gatecli: fix F2, F3`.
- F3: fixed with my own change.
  The proposed injectable step between the `os.Stat` and the `os.Chtimes` was not made.
  `storePrebuilt` no longer does a separate `os.Stat`, which left no window to inject into, so `refreshStored` is the small function the review's alternative names.
  `TestStorePrebuilt` gained two rows: `RefreshStoredReportsAMissingEntryAsNotRefreshed` and `RefreshStoredMarksAnExistingEntryUsed`.
  The missing-entry branch is now also reached by the `EmptyStore` row.
  Same commit as F2, since both change the same function.
- F4: fixed as proposed.
  `TestVerifyGate_RoundScenario` gained a plan_verify failure record scenario asserting the pass of `go test -tags integration -run '^TestB$' example.com/m/b && ` plus the derived command, with nothing after it.
  Commit: `loomshed: fix F4`.
- F5: fixed as proposed.
  `pattern/PATTERN-test-speed.md` states the content-named exported path and that `lyx gate test` gives `go test` no per-invocation value.
  Commit: `pattern: fix F5`.
- F6: fixed as proposed.
  The `CLAUDE.md` push-rule bullet breaks before each added clause.
  Commit: `claude: fix F6`.

Gate runs: `lyx gate test --tags integration` over `./internal/impactset`, `./internal/gatecli`, `./internal/loomshed` and `./internal/pattern`, all green.

## Self-check

- fix-code-correctness, LOW: no test reaches the re-`os.Stat` branch of `refreshStored`, and the re-stat only narrows the prune window.
  Not changed.
  The reviewer's own proposal is the re-stat, and reaching the branch needs an injected step between two adjacent calls; a prune that read the old modification time can still remove the entry after the re-stat, which the review's F2 text already names as the residual window.
- fix-proposal, LOW: `mill-config.yaml:123` `done_gate` still runs the untagged tier before the integration tier.
  Not changed.
  It is the retired mill tool's config, outside the verify gates the board entry names, and no `location` of the review points at it.
- fix-consistency: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
