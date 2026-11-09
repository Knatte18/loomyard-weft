# The local dobby starts a pass on a run while adopt is taking it over

`adopt-old-dobby-starts-pass` · Note · bug, dobby, batten

## Seen

On 2026-11-09 the first `lym adopt --all` held two runs, but dobby started `batten run terminal-titles-hub-project --continue-pass` during the adopt, before the owner was written.

## Idea

Adopt holds every run under its subjects before its reconcile pulls them, and dobby starts no pass on a run whose adopt is in progress.