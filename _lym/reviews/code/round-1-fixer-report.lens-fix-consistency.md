```yaml
lens: fix-consistency
outcome: reported
```

Task: check the change set through the fix-consistency lens (siblings and twins that should agree with the change).

No BLOCKING findings. Two MEDIUM and one LOW.

- **MEDIUM — twin left as it was** (`internal/landingshed/prreview.go:423`, also `internal/loomshed/samplersession.go`, `internal/shedverbs/loopwait.go:318`).
  - F4 was fixed only in `parentlock.Waiter.pause`.
  - `prreview.go:423` has the same `case <-changes:` with no `ok` check.
  - A closed watcher channel would make the PR-Review wait wake at once on every pause, the defect F4 names.
  - The Sampler-Session and `--until-stop` waits are built the same way, per `pattern/PATTERN-daemon-wakeups.md:23-25`.
  - Evidence: my grep over non-test code shows that the only `<-changes` and `<-events` sites that check `ok` are the two in `acquire.go` and one in `reedcli/watchdog.go`.
  - I did not read the other files, so the sites may differ in a way that matters.
  - The review scoped F4 to `acquire.go:76`, so leaving them may be intentional. Say so in the fixer-report.

- **MEDIUM — new behavior not stated in the pattern** (`pattern/PATTERN-daemon-wakeups.md:13`).
  - The parent lock wait now drops a closed watch and falls back to its poll backoff.
  - It also gained a `Waiter.Watch` seam. The pattern entry only describes the event wake.
  - Another bullet in that file (line 41, "One event source") may already cover the closed-watch behavior.
  - The reviewer rejected this file as consistent before the fix, and the `acquire.go` header still fits. The text may still be fine; check it.

- **LOW — garbled sentence** (`internal/landingshed/finalize.go`, doc comment of `judgeFixerChange`).
  - The rewritten text reads "An entry whose Baseline is HEAD has no change, a change within the caps, one a fix review approved at this HEAD, and one the operator accepted at this HEAD let Finalize go on, and the record keeps HEAD as Judged."
  - It runs the "has no change" clause into the list of things that let Finalize go on.
  - Suggested wording: "An entry whose Baseline is HEAD has no change. That entry, a change within the caps, one a fix review approved at this HEAD, and one the operator accepted at this HEAD let Finalize go on, and the record keeps HEAD as Judged."

Checked and consistent:
- The `doc.go` additions (F5 resume-at-front, F6 pause) agree with the `Finalize.Call` doc and the `doc.go` Bound lines.
- The `Store.Covered` field comment now matches the `TryAcquire` doc.
- `Waiter.Watch` is a new optional field, and the other `Waiter` users are unaffected.
- The `Acquire` doc matches the closed-watch behavior.

Tool calls:
1. Ran one grep over `pattern/PATTERN-daemon-wakeups.md` and the non-test Go code for closed-channel handling and the other `<-changes` sites. My context held neither the pattern file nor the twin wait loops.
