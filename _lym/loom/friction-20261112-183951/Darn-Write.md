# Building one coverage map by hand with `lym gate testmap`

Run from the task worktree, `lym gate testmap internal/adoptcli` refused because the worktree is not the prime,
and its way forward named `/home/knatte/Code/loomyard-LYMHUB/loomyard-weft`.
Run there, it refused again: that directory is the prime's weft and holds no `go.mod`.
The call works from the prime's code worktree, `loomyard`, which neither refusal names.
The way forward reads `hubgeom.PrimeWorktreePath`, which returns the weft path, where a code path is needed.
The task allowed one build and no repeats, so a refusal whose way forward fails nearly cost the pilot its one run.
