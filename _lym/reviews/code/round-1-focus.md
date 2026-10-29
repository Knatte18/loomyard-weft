---
round: 1
focus:
  - "internal/gatecli/counters.go `observeTest` and `Failures`: settle whether a leaf failure is always recorded with its cause, across a failing test that prints an error line, a panic, a test whose output arrives after a status line, and a parent and subtest of the same name prefix in one package; check the decision record's D1 acceptance cases against `counters_test.go`."
  - "internal/gatecli/counters.go: settle whether `firstLines` and `failures` are guarded by the same lock as the rest of the counter's state (`handle` runs under the line reader's lock, `Failures` takes it), and whether `Failures` returns a slice a later event can mutate."
  - "internal/adoptcli/check.go `tierNotice`: settle against D3 that the first line stays the summary, the list is capped at ten with a closing `and <n> more`, the `suite.log` path follows, a green tier's notice is unchanged, and a failed tier with no recorded failures (a build failure, a VSTest tier) still reads sensibly; check `check_test.go` covers each."
  - "internal/gateslot/runlog.go and internal/adoptcli/check.go: settle that the new `failures` key is optional in both records (`omitempty`, tolerant read per PATTERN-tolerant-records) and that the docs in `internal/gatecli/doc.go`, `internal/gateslot` and `internal/adoptcli/doc.go` describing the run record, counters and notices match the code."
  - "internal/creelengine/server_test.go hook-wait restart subtest: settle that the two-waiter supersede sync proves registration without a real-time wait, that no spawn seam remains on the waiters, and that the subtest still exercises the reconnect across a daemon restart it was written for."
  - "internal/creelcli/cli_test.go EXTERNAL row: settle that the TTL-expiry and removed-receiver steps step the clock through `creelfake.Clock.AdvanceByWaitsWhere` until the verb returns, with no snapshot wait count left, no unbounded loop if the verb never returns, and no real-time sleep added (PATTERN-test-speed)."
  - "Scope: settle that the diff leaves the creel daemon's production shutdown path and other creel tests untouched, per the decision record's Out list, and that the commit messages' claims match the diff."
---

The diff is two commits: the adopt check's failure naming (`gatecli`, `gateslot`, `adoptcli`) and the two creel test fixes.
Entries name where to look; none states a verdict.
