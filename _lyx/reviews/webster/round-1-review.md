---
verdict: APPROVED
---

# Round 1 review

Range: `git diff $(git merge-base main HEAD)..HEAD` (parent `main`; no `_lyx/loom/rework` round exists).
The publish-failure section is `none`.
`lyx gate test` over `internal/parentreview`, `internal/shuttleengine/...`, `internal/loomcli`, `internal/loomengine`, `internal/shedrecipe`, `contracts/...` and `internal/testkit/...` passes, and so does `cmd/lyx`'s test-budget test.

No finding is raised.
Verdicts on the focus directive's questions:

- **Cards 3 and 5, the `gate` body.**
  No `PassAtCap`, `logPassedAtCap` or `newPassAtCapGates` remains in `internal/` or `contracts/`.
  The remaining `pass_on_cap` hits are the engine-level `PassOnCap` of other gate entries, which card 5 leaves unchanged, and the refusal on `parent-review`.
  The ping case sits after the verdict, expiry and hold checks in the open-request-with-no-verdict branch only, so it runs only while a delivered round is pending.
- **Card 3, keep-alive pings.**
  `pingDue` caps at `maxKeepAlives` (3).
  `Store.RecordPing` persists the count and clock in `delivery.json` under the round lock, so a restart cannot reset it.
  A ping never fires on a settled round or after a verdict, and `final` never pings.
- **Cards 12, 13 and 14, wait loop.**
  The 5-second throttle gates only the pending re-evaluation branch with no new arrival; arrivals, the restored-boundary branch and unsent re-prompt retries stay per tick.
  The off-boundary probe branch runs only with `gateAtBoundary` false, once per `gateRecheckEvery`, and `NewExpiryProbe` reads through `peekLatest` with no lock, no directory creation and no write.
  An expired entry finalizes `done` through `finalize`, which reads each `Final`; a probe error logs a Warn and counts as not expired, so the probe cannot end a wait early.
- **Cards 7, 8 and 9.**
  `PrepareRound` reuses a latest round with a nil `Request`.
  `RecordVerdict` returns `ErrEmptyReviewFile` for a missing, unreadable or whitespace-only file on either verdict kind.
  The plan gives these three cases one message, so a single message is conformant.
  `RecordDelivered` checks for a verdict inside the locked update, writes nothing and logs nothing on a settled round, and `reviewDeliveredVerb` maps that to exit 0 with `settled: true`.
- **Cards 6 and 10.**
  The last-gate refusal sits in `resolveRowGateSpec`, which every gate-list builder (`resolveGateSpec` and the burler row path) goes through, and the empty-list guard precedes the `spec[:len(spec)-1]` slice.
  Every review refusal that names a status command takes `statusWay`, which `reviewStatusWay` sets to `lyx batten status <slug>` for a slug and `lyx loom status` for none.
- **Per-card mechanical checks and test economy.**
  The Delete targets of card 5 are gone and the `fs` import is dropped from `review.go`.
  New top-level tests stay within `cmd/lyx/testdata/test-budget.yaml`, and each covers a behavior no existing test covers.
