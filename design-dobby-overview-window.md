# A dobby overview window per beam: alive, runs driven, tasks waiting and why

`dobby-overview-window` · Note · feature, reed, batten, workspaces

## Seen (2026-10-25)

Nothing shows what a dobby does or whether it lives.
Its log, `<weft>/.lyx/beam/markers/dobby.log`, stays empty unless a tick fails, and its `state.json` records only pass times, plus every run it ever saw, long-landed ones marked `full_pass_owed`.
`lyx beam` has no status verb.
A released task with no dependencies (`umbrella-anchor`) sat unstarted for minutes beside a live dobby, and nothing said why.

## Idea (operator, 2026-10-25)

Every beam, the prime included, gets its own reed window, a dobby overview: whether its dobby is alive, the runs it drives with their row and last pass, and each task that waits, with what it waits on (dependencies, `max_runs`, a pause, a refused start and its backoff).
The same view is printable once, such as by `lyx beam status`, and dobby writes one log line each time it starts a task or gives a run a pass.
