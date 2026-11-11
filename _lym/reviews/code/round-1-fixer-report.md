## Changes

All six findings were fixed as proposed, in the working tree of this worktree, and committed.
`lym gate test` over `internal/webstercli`, `internal/websterengine` and `internal/loomcli` passed (tier 0, none failed).

- **F1** (`internal/webstercli/ownership.go`): `orphanedMasterRefusal` returns the refusal and a `runEnded` bool that is true only for the free-lock case.
  `refuseOrphanedMaster` puts `run_ended: true` into the `ErrFields` map only then.
  The test gained a probe-error row (run.lock is a directory) asserting the re-run way forward and no `run_ended` key.
- **F2** (`internal/webstercli/ownership_test.go`): a subtest of the existing test builds the tree through `addVerbs` with a free run.lock and a Master strand name.
  It runs the six bracket verbs and asserts a `run_ended` refusal, then `status` and asserts none.
- **F3** (`internal/websterengine/verifygatereport.go`): `renderVerifyGateFindings` has a `len(r.Failures) == 0` branch.
  It writes the exit code and "named no failing test", then the log location, and returns without the empty list and the card hint.
  A tail-only case was added to `TestRenderVerifyGateFindings`.
- **F4** (`docs/overview.md`, `internal/loomcli/decision.go`): one sentence in each says a Webster Master session of a run that is not running gets no notice and is named under `notice_held_back`.
- **F5** (`internal/websterengine/verifygate_test.go`): rows for the `gate.Attempts` fallback, a run-level findings text cut on a rune boundary, an output tail cut on a rune boundary, a dirty tree on a later arrival carrying no command, and a named-test failure keeping its failures with no tail.
  The last two are rows of the former `TestVerifyGate_FailureNamingNoTestKeepsTheOutputTail`, now a table named `TestVerifyGate_ReportKeepsWhatTheFailedArrivalObserved`.
- **F6** (`internal/websterengine/verifygate.go`): an empty `LogTail` renders "its output was empty" in place of the "output tail:" clause, with a test row.
- **Self-check fixes**: the `internal/webstercli/doc.go` "Run ownership" section now names the probe-failure refusal.
  The `verifyGateStuckFailure` doc comment now says the output may be empty.

- **Gate repair**: three comment lines of 120 characters or more, in `internal/webstercli/doc.go`, `internal/webstercli/ownership.go` and `internal/websterengine/verifygatereport.go`, were split into shorter sentences.

- **Gate repair, help cap**: F4's added sentence pushed the `decision add` Long help to 2081 bytes, over the 2000-byte cap that `TestCLIForm_EveryCommand` enforces.
  The help now lists `notice_held_back` in the existing key list and uses a shorter example, so it is under the cap.
  The sentence explaining the key stays in `docs/overview.md` and `internal/loomcli/doc.go`.

## Self-check

- fix-consistency, LOW, `internal/webstercli/doc.go` run ownership section stale after F1: fixed as proposed.
- fix-affected-set, LOW, same `doc.go` sentence: duplicate of the above, fixed with it.
- fix-code-correctness, LOW, `verifyGateStuckFailure` doc comment stale after F6: fixed as proposed.
- fix-code-correctness, LOW, the `addVerbs` subtest checks only `status` on the non-bracket side: disputed, `every fix worse` (see Disputed).
- fix-proposal: no findings.

## Disputed

- `every fix worse`, fix-code-correctness LOW (non-bracket verbs beyond `status`): the premise holds, but running `rebaseline`, `accept-audit`, `restore-plan`, `reset` or `run` in the test executes their real verb bodies against a zero-value wiring (`internal/webstercli/reset.go`, `run.go`), which can mutate state or spawn processes.
  The review asked for `status` as the non-bracket control, which reads only and so leaves the wrapped set pinned by the six bracket rows.

## Deferred

Nothing deferred.

## Rulings

No parent ruling reached me.
