```yaml
lens: code-correctness
outcome: reported
```

**Lens: code-correctness. Cases run per function or branch:**

- `overviewWindowKeys` (`windoworder.go`): correct.
  - A non-overview strand, an empty `PaneID` and a pane the listing lacks are all skipped.
  - An overview strand maps its window id to `Order`.
  - A nil or empty `windowByPane` (listing failed) gives an empty map, which is safe.
- `overviewWindowInsertArgs`: correct.
  - No windows gives nil, the plain form.
  - Keys `{@a:0@0, @b:1@1}`, new order 1: `-a -t @b`.
  - Same listing, new order 0: `-a -t @a`, so the new window lands at index 1 and `@b` shifts up.
  - New order 0 against only a key-1 window: `-b -t` the lowest window.
  - Stale key entries for unlisted windows are ignored, because the loop walks `windows`.
  - Hidden windows count only for the "lowest" fallback, as the record says.
- `openOwnWindowLocked` (`spawn.go`): correct.
  - Add: the new strand's `PaneID` is still empty, so it is not in `keys`.
  - Relaunch: the strand's own old window is in `keys` with its own `Order`, so the new window opens right after it.
  - Replace: `ReplaceStrand` has already removed the old strand, so the old window is absent from `keys`.
    - I ran replacing key 0 (old window still alive at index 0) with a key-1 window at index 1: nothing qualifies, so it opens before the lowest window, which is the old one. It ends up in the right place.
  - Listing-failure path: `windowByPane` is empty, so the placement degrades to "before the lowest window". A relaunched key-1 window can then land before the key-0 one (the first finding below).
- `agentWindowPanes` and `relevantWindow` (`ownwindow.go`): correct.
  - A live agent wins first.
  - Otherwise the lowest-index listed window that holds an overview pane wins.
  - `ending` is applied as `skip`.
  - The empty-string result is the same when no overview exists.
  - The old "listed" check is covered, because the loop walks the listed windows.
- `followTargetWindow` (`follow.go`): correct.
  - Overview rows `idx 0` then `idx 1` keep index 0, and the reverse order also keeps the lower one.
  - Unparsable rows are still skipped.
  - A dead overview pane still counts, as before.
- `refuseWindowMove` (`strand.go`): correct.
  - The anchor-move check runs first, so the order check only sees overview-to-overview.
  - `Order` 0 to 1 is refused with both the remove and add halves in the message.
  - A focus-only change passes.
  - `UpdateStrand` has no callers outside reed in production (grep), so the new refusal cannot break a spawner.
- `addStrandLocked`: correct. Removing the overview loop leaves the duplicate-name refusal in place, so the same explicit name can't be added twice.
- `ReplaceStrand` (`replace.go`): only a doc line was dropped. The select-on-replace behavior is unchanged.
- `ensureRunsStrand` (`runs.go`): correct.
  - `break` inside the `if` exits the `for`, which keeps the first match as before.
  - With only a `loom-status` strand, `runs` stays nil and the default `AddStrand` runs.
  - A stale `runs` strand still goes through `ReplaceStrand`.
  - The `agentname` import is still used.
- `statusStrandAddSpec` and `ensureStatusStrand`: correct.
  - It always returns an overview display with `Order` 1, and `Title` is `c.recipe`.
  - `c.recipe` is set at `arm.go:132` (`resolveSeedRecipe`) before `start` ensures the strand.
  - An empty recipe falls back to the window name `overview`.
  - The constant is `RunsStrandOrder + 1`, derived from the one declaration.
  - `ensureStatusStrand`'s keep, replace and add choice is unchanged, so the D5 healing path is intact.

**Findings**

1. LOW, `design`, `internal/reedengine/spawn.go:264-270` and `:283`. When `listSessionPaneWindows` fails, `windowByPane` is empty, so `overviewWindowKeys` is empty. Every overview window then opens "before the lowest-index window". A relaunched or added key-1 overview window can sit before the key-0 one, and nothing repairs the order. The card names this degradation ("as today"), and it needs a tmux listing failure, so the severity stays low. Evidence: `spawn.go:264-270`, `windoworder.go:152-170`.

No other defect found in the lens.

**Tool calls**
1. `grep` for `UpdateStrand(` callers and `c.recipe`: to confirm that no production caller trips the new order refusal, and that `c.recipe` is set before `ensureStatusStrand`. My context only held the definitions.
