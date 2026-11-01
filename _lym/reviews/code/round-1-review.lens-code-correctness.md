```yaml
lens: code-correctness
outcome: reported
```

Task restated: I ran the code-correctness lens over the functions and branches the diff changes.

**Per-branch list**

1. `fixerSession.judge`, dirty with a turn left (`Spent=1`, `cap=3`, dirty `[".scratch-task.txt"]`): correct. It charges the turn (`Spent=2`, record written) and returns `Findings` from `UncommittedFindings`, non-terminal. The next `check` resets `turnStart` through `chargeTurn`, so the timeout check measures the new turn. The test row "re-prompts naming the paths" covers this.
2. `judge`, dirty with no turn left (`Spent=cap`): correct. It halts `fixerEndDirty` with `dirtyReason`, uncharged. `haltReason` still words it as `(dirty)`.
3. Cap comparison: correct and identical to the failing-gate path, `s.rec.Spent >= s.cap`. `Spent` starts at `s0+1` after the first `chargeTurn`. Each re-prompt charges one turn, and re-prompts stop at `Spent==cap`. That is at most `cap-s0-1` re-prompts, below the gate entry's `Attempts=turnsLeft=cap-s0`, so the seat loop's own attempt cap never fires first and the loop cannot run past the cap.
4. `judge`, `dirtyPaths` error: correct. The error is returned and `check` stores it in `s.err`, which `run` returns, and no turn is charged. A `nil` `g.dirty` seam returns `nil, nil`, so a gate told no dirty check skips the check as before.
5. `judge`, dirty plus an escalate report: the dirty check runs first. This is behavior-equivalent to before, since the old code also halted dirty first. The one change is that an escalate verdict with a conflicted tree is now re-prompted to "commit" before it can escalate (finding F1).
6. `verifyGate.clean`, refactored over `dirtyPaths`: correct. A clean tree returns `("", nil)`, a git error returns `("", err)` with the same wrapped message, and a dirty tree returns `dirtyReason`. Callers outside this diff keep the same behavior.
7. `NewCleanGate`, clean tree: returns `Passed:true`. Dirty tree: returns non-terminal `Findings`. `DirtyPaths` error: returned as a wrapped error, a gate error rather than a session finding. Correct. `_lym` and `.lym` do not trip it, because `verifytree.Verify` already refuses on the same `DirtyPaths` call at the same root.
8. `resolvePlainGateClosure`, `clean` case: correct. It rejects an empty or relative `WorktreeRoot` through `requireAbsRoot`. The `default` error message lists `clean` plus the other five names. A repeated name is still rejected by `resolveRowGateSpec`.
9. `UncommittedFindings`: correct for 0, 1 and N paths. It joins with ", ", so a path containing a comma reads ambiguously, and the list is unbounded (finding F2). Nothing else is wrong with it.
10. `pass_on_cap` on a `clean` entry: a recipe author could set it, and the gate would then let a dirty tree through at the cap without a halt (finding F3). The shipped recipes do not set it.

**Findings**

- **F1, LOW, design.** `internal/landingshed/fixerrun.go:257-264`. A fixer whose report says `escalate` with a conflicted tree still gets the commit-or-delete re-prompt, because the dirty check runs before the report check.
  - A mid-merge tree is dirty and cannot be committed as the escalate wants.
  - The fixer burns charged turns on a dirty re-prompt instead of reaching the escalate verdict, and ends `dirty` rather than `escalate` when the turns run out.
  - Evidence: `judge` orders the dirty check (lines 253-265) before `ParseFixerReport` and the escalate branch (lines 266-277).
  - Fix suggestion: parse the report first, or exempt an escalate verdict from the dirty re-prompt.
- **F2, LOW, design.** `internal/verifytree/verify.go:263`. `UncommittedFindings` lists every dirty path with no cap.
  - The existing stuck reason caps at `dirtyPathsShown=10` through `listShown` (`internal/landingshed/verifygate.go:20,192`).
  - An unignored build directory would put thousands of paths into the re-prompt or findings file.
  - Fix suggestion: cap the list the same way.
- **F3, LOW, design.** `internal/shedrecipe/entries_gate.go:114,122`. `pass_on_cap: true` is accepted on the `clean` gate and would turn it into a let-through gate. Fix suggestion: refuse `pass_on_cap` for `clean` in `resolveGateEntry`, or accept the risk. The shipped recipes do not set it.

**Tool calls:** none. My context already held everything this lens needed.
