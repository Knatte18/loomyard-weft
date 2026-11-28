```yaml
lens: fix-affected-set
outcome: reported
```

Task: fix-affected-set lens over the fix's change set.

**Unchecked findings:** none of the set-premised ones (F1/F2/F3 reset-advice sites, F11 callers, F13 callers, F15 sandbox sites, F17 move-aside sites). The others (F4–F10, F12, F14, F16) are single-site premises and I did not re-search them.

1. **LOW**: `internal/fabriccli/doc.go:16`, a sibling of the F1/F3 change that was left as it was.
   - It still says `--reset` "refuses a hub holding a pair or a code commit origin lacks".
   - The fix widened the refusal to unfinished teardowns, uncommitted or untracked changes and a stash.
   - The fixer updated the `--reset` sentence in `fabric.go`'s help and in `fabricengine/doc.go:602-603`, but not this twin.

2. **LOW**: `internal/fabriccli/doc.go:27-31`, a sibling of the F2 change that was left as it was.
   - It says a clone killed before its Bolt checkout leaves a hub where "A plain re-clone into the same directory names [the `--reset` clone] too".
   - `existingHubError` now names the move aside instead when that hub holds work origin lacks.
   - It now also names `--reset` for any other unfinished hub, such as one with a Bolt checkout but no scratch directory or committed anchor.
   - `fabricengine/doc.go` got the new sentence ("It names it for any hub a clone did not finish that holds no work origin lacks"); this file did not.

**Covered:**
- **F11:** both non-test callers of `SettleGonePairTeardown` (`fabric.go`, `cancelcmd.go`) match the new signature.
- **F17:** both refusals that print the move go through `moveAsideWayForward`, and the grep found no other `.old` move site.
- **F15:** no other sandbox scenario I found expects the old refusal behaviour. `sandbox/build.cmd -reset` is the sandbox tool's own flag; I did not read its removal code.

**Outside this lens:** the removal's origin task-branch delete (`deleteTaskBranchAtTip`, `remove.go:610`) still counts other local branches as holding the tip. This is the same hazard F13 fixed for the sweep only.

**Tool calls:**
1. Grepped `SettleGonePairTeardown`/`SettleTeardown` callers; my context had only a pre-change search that did not cover docs and tools.
2. Grepped `--reset` and move-aside sites across `internal`, `tools` and `docs`; my context had no such search.
3. Grepped `checkUnlandedRemoteTip` and local-tag wording; I had only the function body, not its callers.
4. Read `fabriccli/doc.go:12-32`, a `SANDBOX-CORE-SUITE.md` excerpt and the sandbox reset callers; none of these was in my context.
