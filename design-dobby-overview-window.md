# The overview window shows its worktree dobby: alive, runs driven, tasks waiting and why

`dobby-overview-window` · Task · feature, reed, batten, workspaces · running · Code-Burler-Review

## Seen (2026-10-25)

Nothing shows what a dobby does or whether it lives.
Its log, `<weft>/.lyx/beam/markers/dobby.log`, stays empty unless a tick fails, and `lyx beam` has no status verb.
A released task with no dependencies (`umbrella-anchor`) sat unstarted for minutes beside a live dobby, and nothing said why (issue #872).

## Idea (operator, 2026-10-25)

No window of its own: the overview window each worktree's reed session already has (the runs overview, `internal/orchcli/runs.go` and `internal/battencli/overview.go` over `internal/statuspane`) shows the dobby of that worktree, in the prime and in every beam.
It shows whether the dobby is alive, the runs it drives with their row and last pass, and each released task that waits, with what it waits on (dependencies, `max_runs`, a pause, a refused start and its backoff).
The same view is printable once, such as by `lyx beam status`, and dobby writes one log line each time it starts a task or gives a run a pass.
