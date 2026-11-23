# Pin the goto hook's stop of a recorded strand on an unwired receiver

## What changes

`orch-autonomy-mode` (`30ed1ee58`) landed with five review findings in its "To check by hand" list.
That same commit had already fixed and tested all five; only the review ledger still listed them as open.
On the base:

- `goto-stop-reads-unwired-reed-runner`: `prReviewGotoAdmitted` resolves its runner through `stepStopRunner` inside the stop, at call time,
  and `stepStopRunner` builds one when the receiver holds none.
- `clear-review-session-keeps-parent-review-asked`: `ClearReviewSession` clears `ParentReviewAsked`, and `TestClearReviewSession` pins it.
- `dobby-run-notices-unstamped-autonomy`: dobby's run notices are stamped through `orchcli.WithRunAutonomy`,
  and `TestHubDobbyDeps_RunNoticePostCarriesTheRunsAutonomy` pins it.
- `goto-hook-wiring-untested` and `halt-notice-call-sites-untested`: `TestGotoWiring_HandsTheGotoPathsThePRReviewResetHook`,
  `TestGotoSpec_AdmittedHookClearsThePRReviewRecordOnlyForThatRow`, `TestNotifyRunParent_CarriesTheRunsAutonomy`
  and `TestHaltNoticeSeams_CarryTheRunsAutonomy` pin the wiring and each call site's stamp.

One case stayed unpinned: no test drove the goto hook over a record that names a strand, on a receiver wired as `lym loom goto` wires it, with no shuttle runner.
That is the case the blocking finding named.
`TestGotoSpec_AdmittedHookClearsThePRReviewRecordOnlyForThatRow` (`internal/loomcli/gotohook_integration_test.go`) gains that row.
The hook builds its own runner for the stop.
With no reed session up, the stop fails, and the hook returns the `could not stop the recorded PR-Review session` error and leaves the record unchanged.
The existing rows now compare the whole record, not three of its fields.

No production code changes.

## Design decisions

- The entry is labelled `bug`, but no bug remains on the base, so the parent recorded a `No bug fixed` decision:
  this run adds the one row and changes nothing else.
- The row runs at tier 1 without a reed session, so it pins the stop's error path rather than a stop of a live strand.
  A live stop needs a tmux server (tier 2), and `clearPRReviewOnGoto`'s own table already covers the clear after a successful stop.

## Merge danger

**Door:** two-way
**Blast radius:** loomcli
