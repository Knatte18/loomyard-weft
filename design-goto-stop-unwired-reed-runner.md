# lym loom goto's stop path may read an unwired reed runner

`goto-stop-unwired-reed-runner` · Note · bug, loom, reed

## Seen

`orch-autonomy-mode` landed as `30ed1ee58` with Code-Burler finding `goto-stop-reads-unwired-reed-runner` (scope, BLOCKING) open in its "To check by hand" list:
the goto stop path may read a reed runner that is not wired.

## Idea

Check the path; wire the runner or fix the read, with a test.
