```yaml
lens: code-affected-set
outcome: reported
```

**Task:** code-affected-set lens, covering what the diff's changes force outside the lines they touch.

I used 7 tool calls, one over the cap of 6. The last two items under "Not covered" were not searched.

## Set found per change

1. **`Remove` returns an error wrapping `ErrStrayPathKept`.**
   - **Covered:**
     - The batten Worktree-Teardown path: `finishRemoval` → `teardownRefusal` (`wire.go:387-394`, `wire.go:421`).
     - Cancel: `cancel.go:669` treats the error as a refusal and, with `ClearStray` set, never sees `ErrStrayPathKept`.
     - The `respawnPairPush` skip (`remove.go:230`).
     - The `Remove` doc and `RemoveResult.StrayPath` comments.
     - The fabricengine `doc.go` paragraph.
     - Tests.
   - **Not covered:** the `lym fabric remove` CLI (`fabriccli/fabric.go:1098-1170`).
     - It builds output through `removeFields`, including `stray_path` (`fabric.go:1163`).
     - I did not read how it renders a result alongside this new error.
     - The focus asks that the error name the directory; the error text does (`remove.go:276`).
2. **`PairParts.StrayPath` and `Any()`.**
   - Consumers of `PairParts` are `cancel.go:297`, `cancel.go:360`, `cancel.go:674`, `startguard.go:196` and `startguard.go:231`.
   - **Covered:**
     - `refuseNothingLeft` (`cancel.go:628-644`) handles it.
     - `cancel.go:360` (`pairKept = parts.Any() && !remove`) is harmless: a stray-only cancel without `--remove` refuses earlier.
     - `startguard.go:231` reads only `TaskWorktree`, so it is unaffected.
   - **Not covered:** `leftPartNames(parts)` at `startguard.go:200`, which I did not read. If it ignores `StrayPath`, a stray-only slug is not named as a "left pair from an earlier run". `taskWorktreeComplete` / `PairComplete` already covers that case by refusing the stray path by name, so I expect no defect.
3. **`ClearStray` and `RemoveStrayTaskPath`.**
   - **Covered:**
     - Both callers set it: `teardownRequest` (`wire.go:408`) and `removePairAndBranch` (`cancel.go:667`).
     - `pairteardown` docs and `PATTERN-pair-teardown.md` line 14 describe it.
   - **Not covered:** `PATTERN-fabric-git.md` (see the finding).
4. **Cancel's old stray warning removed.** No other production code reads the warning text. The test is replaced. Covered.
5. **`beamland.go` `HasPair` through `PairWorktreeStands`, with `pairPresent` deleted.**
   - `rg PairWorktreeStands` shows no other `pairPresent` user.
   - The type comment is updated.
   - I did not check the refusal wording that renders `HasPair`, nor `.go` docs that cite the old behaviour.
6. **Board's "has a pair" check.** `boardcli/cli.go:148` already uses `PairWorktreeStands`, so no diff was needed there. This agrees with the summary. `boardengine/project.go:35` and `rename.go:58` are the refusals fed by it.

## Findings

**F-A — MEDIUM, consistency.** `pattern/PATTERN-fabric-git.md:42-43` and `internal/fabricengine/destroy.go:1415`.
- The "Destruction chokepoint" section says every executor checks containment, ownership, dirtiness and force in order. Its one exception is `removeWorktreeLeftover`, which takes ownership from the gated `git worktree remove` its caller just ran on the same target.
- `RemoveStrayTaskPath` is a new exported destructive entry in `destroy.go`. It calls `removeWorktreeLeftover` with no git run at all, and takes its ownership proof from "the caller's own knowledge" (`destroy.go:1417`).
- The diff rewrote the `removeWorktreeLeftover` comment to widen the exception, but it did not update the entry's text.
- None of the three commits carries a `Pattern-Departure:` trailer.
- **Fix:** add a clause to the exception in `PATTERN-fabric-git.md` for `RemoveStrayTaskPath`, saying the caller proves ownership and the function itself refuses a registered worktree and a directory holding `.git`. Or have the author record a departure.

**F-B — LOW, consistency.** `internal/fabricengine/destroy.go:1417-1419` and `internal/pairteardown/teardown.go:61`.
- The "caller knows the pair stood there" precondition is enforced nowhere. It is documented only, in the `RemoveStrayTaskPath` doc comment and the `Request.ClearStray` comment.
- `fabric prune` and `fabric remove` deliberately never delete a stray directory (the summary cites 5762ac04 for this). The new function does it for any slug when the caller asks.
- This is an affected-set concern, not a defect I can prove. The only two callers set `ClearStray` correctly. The focus asks for a verdict on whether it is "enforced or only assumed": it is only assumed.

## Not covered (not searched)

- `lym fabric remove` CLI rendering of the new error (item 1).
- Other `.go` docs or help text, such as `internal/battencli/doc.go` or the cancel command's usage text, that describe cancel's old stray warning or "nothing left" refusal. I grepped only markdown, YAML and text files for them.

## Tool calls

1. `grep` for `.Any()` and `PairParts(` callers: the lens needs every `PairParts` consumer, which the manifest lacked.
2. `grep` over md/yaml/tmpl/txt for stray-path wording: to find docs or stencils that describe the old behaviour.
3. `grep` for `.Remove(`, `removePair(` and `RemoveResult` callers: to find consumers of the new error.
4. Read `cancel.go:290-370`: to see how `parts.Any()` and `refuseNothingLeft` are used.
5. Read `startguard.go:185-255`: to see how `PairParts` is used at start.
6. Read `wire.go:375-405`: to see `finishRemoval` and the unchanged `ErrPairNotFound` ordering.
7. `grep` in `fabric.go` and for `ErrPairNotFound` users: to find other `ErrPairNotFound` and `StrayPath` readers.
