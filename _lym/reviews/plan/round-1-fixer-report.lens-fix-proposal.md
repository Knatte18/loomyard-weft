```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge each review proposal the fixer carried out as proposed (F2–F12), looking for a concrete case where the proposal is wrong or breaks something elsewhere.

**[MEDIUM] Card 8, new `TestRun_WorktreeGoneExits` row (F2): the step as written cannot be carried out.**
- **Changed text:** "a worktree removed after tick 1 has ended, from inside a `Deps.Sleep` wrapper that removes it before returning the harness's watch event, with no probe due".
- **Problem:** `Deps.Sleep` returns nothing, so it cannot return a watch event. Watch events arrive only on the `Watch` channel.
- **Evidence:** `Sleep func(ctx context.Context, d time.Duration)` at `internal/dobbyengine/loop.go:47`. `daemon.sleep` (`loop.go:337-367`) takes a pending event first, skipping the timer `Deps.Sleep` altogether (339-341). On a wake it calls `Deps.Sleep` again only for the `wakeFloor` rest (358-360).
- **Effect on the row:** the row inherits the setup in which tick 1's hook sends one watch event. With that event pending, the wrapper runs only through the `wakeFloor` call. On a wake with no event pending it can run twice, so a removal that is not guarded to one call is ambiguous.
- **Way out:** state the row's mechanics. In this row tick 1's hook sends no event. The wrapper removes the worktree on its first call only, then sends one value on the harness's `Watch` channel and returns.

**[LOW] `pattern/PATTERN-test-speed.md` Gate keys row added by card 11 (F10): the table's lead sentence no longer holds for the new row.**
- **Changed text:** "Gate keys table gains the row `adopt`, `lym adopt check`, `all`".
- **Problem:** the table opens with "a site runs `lym gate test --wait --gate &lt;key&gt;`; no site spells a tier list…". `lym adopt check` runs the gate in-process through `gatecli.RunGate`, not through that command. So the new row makes the lead sentence false for this site.
- **Evidence:** PATTERN-test-speed.md line 27. Card 12 line 28 says `RunGate` runs `runTest` in-process.
- **Way out:** the row's Site cell says "`lym adopt check`, in-process through `gatecli.RunGate`", or the lead sentence admits an in-process run of the same key.

Checked with no finding:
- **F3 and F9:** the reordered rows and the moved assertions hold. With `Anchor: "."` the `.lym/beam` directory sits under the root.
- **F4, F6, F8, F11, F12:** no concrete case breaks these proposals. Under `--wait`, `acquireSlot` still ends before any tier on a cancelled context, which the "refused key" wording omits but which never sets `Running`.
- **F5:** `state.ReadJSONNoMkdir` maps an undecodable file to `ErrDecode` like `ReadJSON`, so `ReadCheck`'s read-as-absent still works.
- **F7:** `lock.AcquireReadLock` opens the lock file through `flock` without creating directories, so a removed `.lym/beam` gives a read error and not a recreated stub. That matches the card's read-error-means-gone branch.

Tool calls:
- Read `lock.AcquireReadLock` and the `Deps.Sleep` field to check F7's no-mkdir claim and F2's seam shape. My context had `state.ReadJSONNoMkdir` but not the lock function or the Sleep signature.
- Read `daemon.sleep` in `loop.go` to see how a watch event and `Deps.Sleep` interact for F2. My context showed only the loop's call to it.
- Grepped `hubforge` and `hubgeom` for where the beam directory sits, to check F9's "under the row's root" assertion. My context lacked the beam directory's placement.
