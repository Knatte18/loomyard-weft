---
round: 1
focus:
  - "Scope against the Goal's first idea (a cancel never waits on a step held by its own pause): the diff changes `gitrepo.HoldRemoteRefs` and `pauserecord.Network`/`Admit`, and nothing in `internal/battencli/cancelcmd.go` orders the cancel's pause against the running step. Settle whether the diff makes a cancel end the step or skip the pause before it, or only narrows one lock-holder path, and whether the Open-IDE step wait described under Seen can still occur."
  - "`pauserecord.Admit` and the process-wide `admissions` counter in `internal/pauserecord/hold.go`: settle whether one admission lets every goroutine's `Network` call pass the gate (including calls unrelated to the lock holder), whether `Admit` called while a pause already covers the process can wait forever or leave a stale in-flight marker, and whether the release order in `HoldRemoteRefs` (lock release, then gate leave) holds on every error path."
  - "`removeRunRecords` in `internal/battencli/cleanup.go`: settle whether skipping the commit when the records HEAD no longer tracks the run leaves a stale `_lym/shed/<slug>` directory or the cancel marker in place, and whether the `os.MkdirAll` for the status lock's directory named in the Goal's cause section is addressed (an empty recreated directory is the stated trigger of the failed `git add`)."
  - "Callers of `removeRunRecords` other than `cancel --remove` (cleanup passes): settle whether the changed commit condition alters their behavior, in particular the re-run that finishes a removal whose commit failed earlier, and whether `deps.Committed` is set for every `cleanupDeps` construction (`internal/battencli/cli.go`, `cancelcmd.go`)."
  - "Test coverage: settle whether the new tests in `cleanup_test.go`, `hold_test.go` and `remoterefslock_integration_test.go` cover each added behavior (already-removed records, admitted network call, lock holder passing the gate) without duplicating an existing test, and whether any behavior of the diff has none."
  - "Commit messages, `PATTERN.md` and the `doc.go` changes: settle whether the `PATTERN-remote-refs-lock` text matches what `HoldRemoteRefs` now does and whether the docs the repository names for these changes landed in the same diff."
---

Seeded from the decision record's Goal and the diff stat of the review range.
