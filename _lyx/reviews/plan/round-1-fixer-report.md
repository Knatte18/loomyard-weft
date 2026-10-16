# Fixer report — plan, round 1

## Changes

- **F1 — fixed differently.**
  The premise holds: card 3 created twin `rig.probeUntilDone` helpers in `internal/creelengine_test` and `internal/creelcli`.
  I did not make the proposed change of running `creelfake.Clock.AutoAdvance`.
  `AutoAdvance` (`internal/testkit/creelfake/creelfake.go:295`) jumps straight to the earliest pending timer as soon as it fires one, without waiting for the woken worker to re-arm its next probe wait.
  A queued inject makes `Store.sweepNeeded` true (`internal/creelengine/sweep.go:146`, `sweepSubjects` counts queued injects), so the sweep timer is pending throughout these rows.
  Right after a probe timer fires, `AutoAdvance` can therefore jump to that sweep timer, more than the 16-second `stallFreshness` past the last reading, and the row's asserted expired stall would be dropped as stale.
  That race leaves the rows flaky.
  Instead I took the review's own fallback, one kit method that both packages call: card 3 now creates `plan:internal/testkit/creelfake#Clock.AdvanceByWaitsWhere`, `func (c *Clock) AdvanceByWaitsWhere(keep func(time.Duration) bool, done <-chan struct{})`.
  It steps by each newly recorded wait `keep` accepts and waits for the next one before stepping again, until `done` closes, and gives up after a real ten seconds as `AwaitWaitsWhere` does.
  Both `rig.probeUntilDone` Create lines are gone, `internal/testkit/creelfake#Clock.AwaitWaitsWhere` is under **Uses:**, the Intent describes the method and how the rows close a `returned` channel and call it with `isProbeWait`, and the ImpactSummary names the kit method.
  I first listed both packages' `isProbeWait` under **Uses:**, but `validate-plan` raised `uses-later-target` because card 6 edits them, so I dropped those two lines and the Intent names the predicate in prose.
- **F2 — fixed as proposed.**
  Card 10 drops `fakeEngine` and `fakeEngine.TrustDismissSequence` from **Edit:**, lists `internal/shuttleengine#undismissableEngine` under **Uses:**, removes the `NoTrustInputs` paragraph, rewrites the empty-sequence row over `undismissableEngine`, and drops "one fake field" from the ImpactSummary.
- **F3 — fixed as proposed.**
  Card 3 drops the new `TestCreelCLI` row about an `inject --wait` after a draft cleared under a hold.
  `internal/creelcli#TestCreelCLI` stays in **Edit:** for the existing row's clock stepping.
- **F4 — fixed as proposed.**
  Card 6's Bound paragraph now says the shorter guard lets a strand absent from reed's persisted state for 10 seconds or more have its held mail returned, where before it had to be absent for at least 30 seconds.
- **F5 — fixed as proposed.**
  Card 11 now names the poll's exits: the first tick where `Runner.AcceptStartupGate` reports true, or where `hubDelivery.Idle`'s answer carries no `no-input-box` stall, ends the poll.
  At `smokePaneWindow` the test fails and prints the strand's last pane capture.

`lyx loom validate-plan` passes on the edited plan.

## Disputed

None.

## Deferred

None.
