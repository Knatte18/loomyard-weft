# runpusher tries to start pushers for removed task worktrees

`runpusher-removed-worktrees` · Task · bug, fabric · done

Imported from [issue #286](https://github.com/Knatte18/loomyard/issues/286).

On `lym board sync` from the prime, runpusher logs one WARN per task worktree that was removed long ago:

`runpusher: the pusher did not start ... resolve main worktree: list git worktrees in "<hub>/<slug>": dotgit: not a git repository: no .git above <hub>/<slug>`

Five such slugs were listed, all of runs that landed and whose pairs were removed through lym fabric. The pusher's run registry keeps entries for worktrees that no longer exist. Expected: entries for removed pairs are dropped at removal (or skipped silently when the directory is gone).