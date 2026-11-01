```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the round-2 changes agree with the parts of the decision record and support log they touch, and find any change whose twin elsewhere was left unchanged.

- MEDIUM — D6, new sentence "Where the move's caller has no result to carry the warning, as a code-branch fast-forward in `ReconcileOrigin`… called from `battencli` or `hibernatecli`, the failure is log-only". This conflicts with the sentence before it, which was not changed: "a foreground verb also reports it in its result". `hibernatecli`'s reconcile is a foreground verb (`internal/hibernatecli/reconcile.go:46`). So are the other `ReconcileOrigin` callers: `orchcli/start.go:285`, `fabriccli/remove.go:212`, `adoptcli/deps.go:67` and `loomcli/landingdeps.go:669`. Under the old sentence each of them would report the warning; under the new one it is log-only. The rule should be restated as "a verb whose result carries the warning reports it; any other move logs only".
- LOW — D6, "A detached sync is a lym process with the same pre-run, so it reaches the seam too." This sentence was left unchanged after F3 dropped the Bolt-sync example. It still claims the detached sync matters to the seam, which F3's premise denies.
- LOW — D5 acceptance, "weft's exclude holds neither; … the wired junction names hold neither". The list now has three entries, since the template default gained `.heddle`, but "neither" still counts two. It should read "none of them".
- LOW — D5 body, "every lym worktree carries the files". This reason for never unseeding a line was left unchanged. It no longer holds for `.heddle`, which D3 removes after each render, so no worktree carries it.
- LOW — Constraints, PATTERN-durable-vs-ephemeral-state: "the rendered files are never tracked and sit outside `.lym`". The `## PATTERN changes` twin was widened to "rendered instruction files and their `.heddle` temporary directory", but this Constraints line was not.

No further tool calls made.
