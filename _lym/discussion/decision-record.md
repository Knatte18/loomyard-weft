## Goal

```markdown
## Goal

Close the four findings `orch-autonomy-mode` (`30ed1ee58`) landed open in its "To check by hand" list.

## lym loom goto's stop path may read an unwired reed runner

### Seen

`orch-autonomy-mode` landed as `30ed1ee58` with Code-Burler finding `goto-stop-reads-unwired-reed-runner` (scope, BLOCKING) open in its "To check by hand" list:
the goto stop path may read a reed runner that is not wired.

### Idea

Check the path; wire the runner or fix the read, with a test.

## ClearReviewSession may keep the PR-Review parent ask after goto

### Seen

`orch-autonomy-mode` landed as `30ed1ee58` with finding `clear-review-session-keeps-parent-review-asked` (design, MEDIUM) open:
`ClearReviewSession` in `internal/pullrequest` may leave the PR-Review parent ask state set after `lym loom goto`.

### Idea

Clear the ask state with the session, with a test.

## Some dobby halt notices lack their autonomy stamp

### Seen

`orch-autonomy-mode` landed as `30ed1ee58` with finding `dobby-run-notices-unstamped-autonomy` (scope, MEDIUM) open:
not every halt notice dobby sends carries its `autonomy:` stamp, so the orch may handle such a halt under the wrong autonomy.

### Idea

Stamp every dobby halt notice, with a test per call site.

## Autonomy mode's goto hook and halt-notice stamps have no tests

### Seen

`orch-autonomy-mode` landed as `30ed1ee58` with findings `goto-hook-wiring-untested` and `halt-notice-call-sites-untested` (scope, MEDIUM) open:
no test pins the goto hook that clears the PR-Review record, nor the `autonomy:` stamp at each halt-notice call site.

### Idea

Add those tests.

```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-23): No bug fixed: the four findings were fixed by 30ed1ee58

Decision: All four findings are already fixed and pinned on the base. This run adds one row to TestGotoSpec_AdmittedHookClearsThePRReviewRecordOnlyForThatRow: a recorded strand on a receiver with no runner, the exact case finding 1 named. It changes nothing else.

Rationale: The run found each finding fixed and tested on HEAD; only that one case is unpinned.

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
