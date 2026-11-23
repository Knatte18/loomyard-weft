# Close the four findings autonomy mode landed open

`autonomy-mode-followups` · Task · bug, loom, landing, dobby, orch, reed, testkit · done

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

