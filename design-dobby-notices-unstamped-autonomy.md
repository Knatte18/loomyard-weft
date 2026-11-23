# Some dobby halt notices lack their autonomy stamp

`dobby-notices-unstamped-autonomy` · Note · bug, dobby, orch

## Seen

`orch-autonomy-mode` landed as `30ed1ee58` with finding `dobby-run-notices-unstamped-autonomy` (scope, MEDIUM) open:
not every halt notice dobby sends carries its `autonomy:` stamp, so the orch may handle such a halt under the wrong autonomy.

## Idea

Stamp every dobby halt notice, with a test per call site.
