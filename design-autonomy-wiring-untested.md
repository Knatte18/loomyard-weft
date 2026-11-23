# Autonomy mode's goto hook and halt-notice stamps have no tests

`autonomy-wiring-untested` · Note · bug, loom, orch, testkit

## Seen

`orch-autonomy-mode` landed as `30ed1ee58` with findings `goto-hook-wiring-untested` and `halt-notice-call-sites-untested` (scope, MEDIUM) open:
no test pins the goto hook that clears the PR-Review record, nor the `autonomy:` stamp at each halt-notice call site.

## Idea

Add those tests.
