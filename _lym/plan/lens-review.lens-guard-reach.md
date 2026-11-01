```yaml
lens: guard-reach
outcome: reported
```

guard-reach lens: tracing each guard, refusal and branch the cards add to a reachable state.

Findings:

1. MEDIUM — Card 10 (D3 lease cancel, D1 split stdout/stderr): the plan does not say how the runner calls the in-process gate tree, and the obvious route cannot reach the lease guard.
   - The card says the runner executes `gateTree()` over `test --wait <args>` with stdout and stderr in separate files, and cancels "the call's context" once the lease passes.
   - The only cwd-seeding entry is `clihelp.ExecuteIn`, and the gate helper already uses it. That entry seeds `context.Background()` (internal/clihelp/exec.go:220-229), so a lease cancel never reaches `runTest`'s `runCtx`.
   - It also sets `cmd.SetErr(out)` (exec.go:223-224), which merges stderr into stdout.
   - Through that route, the TestGateRunner rows "a lease of two seconds that passes while the stand-in holds" and "the lease passes in the slot wait" can only end at the tier timeout, and the `stderr` file stays empty.
   - The card should name the route: `clihelp.RunRootCtx` over `lymcwd.WithCwd(leaseCtx, cwd)`, with `SetOut`/`SetErr` set to the two files.

2. LOW — Card 11 and Card 13 (D1 join check): the tree and dirty-path guard does not cover the plan, so the same staleness can recur unguarded.
   - `_lym` is a symlink into the weft worktree (`gate-run-keeps-cache-warm/_lym -> …-weft/_lym`). `verifytree.DirtyPaths` and `headIdentity` run git in the warp worktree, so they never see an edit to `_lym/plan`.
   - For `card-verify` (keyed by verb, run dir and `card.ID()`), an edited `**Verify:**` line joins and replays the old line's result while the lease holds.
   - The same holds for `lym gate test --card N` after a card's targets change.
   - Neither card names this in its Bound.

3. LOW — Card 11 (D2 inherited-slot branch of `frontDecision`): its state cannot arise in production.
   - The branch needs a strand and a held `gateslot.InheritEnv` together. Every path that holds a slot runs its child without the strand:
     - tier env strips it (internal/gatecli/run.go:689-691);
     - `verifytree` strips it (internal/verifytree/verify.go:382);
     - `cardverify` strips it (card 5);
     - the detached runner strips it (card 11's `launchRunner`).
   - An agent shell cannot set the variable, because of the standing deny.
   - So only a hand-set variable or the test row reaches it. The guard is kept because D2 lists it, but its Intent claims a path that cannot occur.

4. LOW — Card 12 (`longWait` empty-scope rule): a build-blocked tier is misread as an empty-scope tier.
   - The card treats a non-build, non-carried record with an empty `Command` as an empty-scope tier.
   - A tier blocked by a failed build has the same shape: `reportBlocked` writes a record with no `Command` (internal/gatecli/build.go:83-90).
   - So a call whose build failed is silently dropped as "empty-scope" rather than for its own reason.
   - The effect is harmless to the median, but the rule's stated guard does not match the state it catches. Testing `BlockedBy == ""` in the classification would separate the two.

Tool calls:
- grep of `internal/clihelp` execute functions, `runTier`'s record build and `ls -la` of the worktree for `_lym`: my context lacked `ExecuteIn`'s context and stderr wiring, whether an empty-scope record carries a `Command`, and whether `_lym` lives in the warp tree.
- sed of internal/clihelp/exec.go:195-235 and internal/gatecli/tierrun.go:50-80: to confirm `ExecuteIn` seeds `context.Background()` and merges stderr, and how `runTier` builds an empty-invocation record.
