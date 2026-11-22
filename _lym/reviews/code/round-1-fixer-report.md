---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed-differently
    reason: "The proposed rows reuse the orchcli creel-on-fake-delivery harness, whose helpers are unexported in package orchcli, and the testkit may not import hubforge or the CLI packages (internal/testkit/enforcement_test.go), so each package that posts (battencli, loomcli, beamcli) carries its own small daemon helper, as beamcli already did. The hand-back seam is built inline in goDriverDepsFor, so handBackNotify extracts the closure and the test reaches it through goDriverDepsFor over a wired loomCLI."
  - id: F4
    disposition: fixed-differently
    reason: "The two inline RelaunchRequest literals of loomConsumeAttempt and goDriverDepsFor now come from one loomCLI.relaunchRequest builder, and the test asserts that builder's Admitted is non-nil instead of reaching each caller. The goto spec's hook is called with PR-Review over a real pair's record holding autonomy and the ask mark, with the commit and push skipped through FABRIC_SKIP_GIT and FABRIC_SKIP_PUSH; the record holds no strand, since stopping a live strand needs the shuttle and reed config a test hub does not carry."
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed-differently
    reason: "TestRelaunch already holds the Admitted-error row (admit error returned verbatim), so only TestGoto_Guard gained one. The nil PRReviewStanding row ends in the missing-seam error that the session start returns before the wait's judge runs, so it pins the wiring error of a review under autonomy and not judgeParentAnswer's own check, and it needed a wantErrText field with an exemption of the wait-marks assertion."
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
  - id: F10
    disposition: fixed
---

# Round 1 fixer-report

Every finding is fixed, each in its own commit but F3 and F4, which share `internal/loomcli/drivewiring.go` and so form one group.
The gate passes at tier 0 and tier 1 over the touched packages: `pullrequest`, `shedengine`, `landingshed`, `dobbyengine`, `orchengine`, `orchcli`, `beamcli`, `battencli` and `loomcli`.

## Changes

- **F1** `internal/pullrequest/record.go`: `ClearReviewSession` also clears `ParentReviewAsked`, and its doc comment lists the field.
  `TestClearReviewSession` holds a record with the mark true and expects it cleared.
- **F2** `internal/orchcli/notify.go` exports `WithRunAutonomy`, and `internal/beamcli/dobbydeps.go` stamps each run notice with it while `geom.Slug` stays the signer scope.
  `internal/beamcli/dobbynoticestamp_integration_test.go` posts through the prime's dobby and a beam's dobby and asserts the sender, the scope and the stamp.
- **F3** `internal/battencli/noticestamp_integration_test.go` pins `notifyRunParent` for a present pair and an absent one.
  `internal/loomcli/notifystamp_integration_test.go` pins the hand-back notice, reached through `goDriverDepsFor`, and the pre-push check's notice.
  `drivewiring.go` gains `handBackNotify`.
- **F4** `internal/loomcli/arbiterconsume.go` gains `relaunchRequest`, which both loom relaunch sites now use.
  `gotowiring_test.go` asserts the goto spec's and the request's hook are non-nil.
  `gotohook_integration_test.go` calls the spec's hook for PR-Review and for another row over a real pair.
- **F5** `internal/battencli/drive.go`: the doc comment of `haltedRowGuard` states that the batten recipe has no PR-Review row, which is the review's second option.
- **F6** rows added to `TestLoadConfig` (a non-boolean autonomy), `TestPRReview_Wait` (a failing standing read and an unset standing seam) and `TestGoto_Guard` (an `Admitted` error with the move already written).
- **F7** `internal/loomcli/pawldrain_integration_test.go` drives `HoldsCommits` and `WorkRemains` over a real beam, with a draft member and with a child pair.
- **F8** `docs/overview.md`: the Slack hook's autonomous behaviour and the `[beam drained]` line.
- **F9, F10** `//testtiming:keep` lines above the two tests.

## Self-check

- fix-code-correctness, MEDIUM, nil-standing row does not reach `judgeParentAnswer`'s own seam check: disputed, `every fix worse`.
  No row can reach that check, because `checkAutonomySeams` already returns the same error at session start (`internal/landingshed/prsession.go:76`).
- fix-code-correctness, LOW, `HandBack.Notify` assignment unpinned: fixed with my own change, the test now drives the notice through `goDriverDepsFor`.
- fix-proposal, LOW, same finding: merged with the one above.
- fix-code-correctness, LOW, goto-wiring test only asserts non-nil: fixed as proposed, `gotohook_integration_test.go` calls the hook.
- fix-proposal, LOW, same finding and an inline `RelaunchRequest` could return unnoticed: merged; the second part is disputed, `every fix worse`, since a test cannot pin that a call site uses a builder and the builder's `Admitted` is asserted.
- fix-proposal, LOW, drain test's member has no draft status: fixed as proposed.
- fix-consistency, LOW, `prreviewgoto.go` header omits the ask mark: fixed as proposed.
- fix-affected-set: no findings.

## Disputed

- fix-code-correctness MEDIUM (nil-standing row), ground `every fix worse`.
  Evidence: `internal/landingshed/prsession.go:76` and `internal/landingshed/prsession.go:92` call `checkAutonomySeams` before the wait loop, so `internal/landingshed/prreview.go:972` is never the first to find an unset seam.
  Deleting the row would drop the pinned wiring error, and the check cannot be reached by any other row.
- fix-proposal LOW (inline `RelaunchRequest` returns), ground `every fix worse`.
  Evidence: `internal/loomcli/arbiterconsume.go` holds the one builder, and `internal/loomcli/gotowiring_test.go` asserts its hook.
  A test that pins which of two equal-shape literals a call site uses would assert the implementation and not the behaviour.

## Gate repair

The gate refused six comment lines that wrapped at a fixed column or reached 120 characters.
They are re-broken at sentence boundaries in `internal/battencli/noticestamp_integration_test.go`, `internal/loomcli/gotohook_integration_test.go`, `internal/loomcli/prreviewgoto.go`, `internal/pullrequest/record.go` and `internal/pullrequest/record_test.go`.
The gate then refused one remaining wrapped break in the `ClearReviewSession` doc comment, which is rewritten as two sentences.
No behaviour changed.

## Deferred

Nothing deferred.

## Rulings

None reached me.
