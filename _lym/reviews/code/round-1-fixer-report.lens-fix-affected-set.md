```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: no findings. I searched the sets behind F1, F4 and F5, and the fix covers every member I found.

- **F1 (stale "open bound" wording):** no `loadTurnOpenBound`, "open bound", "end of the startup probe" or "never starts or ends" remains in any `.go` or `.md` file. The two other hits for "15 minutes" are unrelated (`internal/creelcli/cli_test.go:786`, `docs/benchmarks/test-suite-prune-rest.md:21`).
- **F4 (stale prose sites):** each of these now names the hold or the skill-load notice:
  - `Run.deadline` (`run.go:346`)
  - `Runner.notifier` (`run.go:56`)
  - `SetNotifier` (`run.go:93`)
  - the `doc.go` lines near 184, 321, 322 and 328
- **F4 (still true, no change needed):** `doc.go:95` says Wait calls the notifier on a held turn end, and that stays correct. `run.go:792` says "one whose turn timed out are skipped", which is still true for a turn with no turn start read. The `SkillLoadTimeout` comment in `spec.go` was updated.
- **F5 (pattern entries):** `loadturn.go` is already in the declared wake list in `cmd/lym/daemonwakeups_test.go:40`. Only the two pattern files name the hold, and both were edited.

I did not check the `doc.go:124` glitch `...since no re-send changes the model.// RunState.NotifiedOffset...`. It looks like it predates this change.

Tool calls (3 greps):
1. Searched for the stale wording from F1 across `.go` and `.md` files. Reason: my context held only the files I had read, not the whole tree.
2. Searched for the F4 comment and doc sites, including `SkillLoadTimeout` and the notifier text. Reason: same.
3. Searched for other files naming `loadturn.go` or the pattern sections. Reason: same.
