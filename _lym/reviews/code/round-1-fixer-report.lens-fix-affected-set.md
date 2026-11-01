```yaml
lens: fix-affected-set
outcome: reported
```

Task: check the fix's coverage of each set-premise finding (fix-affected-set lens).

Unchecked: F1 (decision, excluded), F8 (disputed, no code change), F9 (a single paragraph, not a set).

**Findings**

- **LOW — F7, `internal/gatecli/detachfront.go` lines 474, 487, 494, 605, 699 and 820.**
  - **Summary:** The F7 fix covers the three sites the review named (`replayedBusyLine`, the going message, `abandonedLine`). It leaves six other caller-visible `output.Err` messages saying `gate test:` for a card-verify front.
  - **Evidence:**
    - `grep` of `gate test:` in `detachfront.go` shows these messages: "cannot create the run directory", "cannot wait for the run", "cannot read the run's result", "cannot take the lock of the run directory", "cannot prepare a run under" and "the detached runner did not start".
    - They go to `f.out`, and `f.call.verb` is available at every site. `verbCardVerify` fronts exist (`detachfront_integration_test.go`: `cardCall`, `cardKeyDir`).
    - The review's premise was verb-blind text for card-verify runs. These sites have the same defect and still use `gate test`.
  - **Not findings:** The `logger.Warn` lines are operator logs and out of scope. `detachRefusal` already names both verbs.

- **No finding — F2.** `copyRunFile` has one caller, in `replay`, and it now returns an error there.
- **No finding — F3.** `leaseHolds` is still used where the review wanted strict behaviour: the startup refusal in `runDetached` (`detachrun.go:260`) and two test assertions. `leaseLapsed` is used only in the lease loop (`detachrun.go:369`).
- **No finding — F5.**
  - The `claim` callers are `runDetached`, `claimByFront` (`detachfront.go:747`) and tests, and all take the link path.
  - A transient `process.json.*.tmp` is removed by a defer.
  - The test `settleRunners` globs only `<key>/<run>` directories, so the temp file does not matter.
- **No finding — F6.** `pruneOldRuns` is called once, from `startRun`. Failed-launch runs reach it through the same path.
- **No finding — F7.** The `replayedBusyLine` and `abandonedLine` call sites in tests are updated: `detachfront_integration_test.go:434` and `test_integration_test.go:2051`.

**Calls made**
1. `grep` of `gate test` strings in `detachfront.go` and `detachrun.go`, plus the `copyRunFile`, `leaseHolds`, `leaseLapsed` and `.claim(` callers: my context had no usage lists for the finding-set searches.
2. `grep` of `verbCardVerify` and `runCard` across `internal`: to confirm no other production site builds card-verify messages. It found none outside `gatecli`.
