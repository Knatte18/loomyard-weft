---
round: 2
focus:
  - "`internal/fabricengine/clone.go`, `hubWorkOriginLacks` and `checkResetTarget`: is there any state held only on this machine that `--reset` can still delete? Consider local-only tags, pending-teardown records, and the prime code store's uncommitted or untracked files, stash and detached HEAD. Does the refusal also avoid blocking a hub that holds no such work?"
  - "`internal/fabricengine/clone.go`, `existingHubError` and `checkResetTarget`: for a whole hub, a partly cloned hub (including one with a Bolt checkout but unfinished anchor or layout steps) and a hub holding work, does the printed way forward work as written? Check what moving a hub does to its linked worktrees, and what happens when the target name already exists."
  - "`internal/fabricengine/cleanupremotewarp.go`, `deleteLeftoverUnderLock` and `checkUnlandedRemoteTip`: after the sweep deletes a branch from origin, is its tip still reachable from something that outlives this machine? Is the keep-the-branch path on a failed tag push tested?"
  - "`internal/fabriccli/remove.go`, `settleGonePair`, with its wiring in `internal/fabriccli/fabric.go#runRemoveWithFlag` and `SettleGonePairTeardown`: does the result report the mutations made and any stray path kept, is settling limited to `--remote`, and do tests hold these behaviours and the re-run command it prints?"
  - "Do the tests fail when each fix is reverted? Check the catch-up call site in `internal/battencli/catchup.go#fetchRunForCatchUp` for a slug with no pair, and a refused board-ref push after the records push landed during a one-repository clone."
  - "Do `internal/fabricengine/doc.go`, `tools/sandbox/SANDBOX-FABRIC-SUITE.md` and `_lym/landing/summary.md` match the code? Check who calls `SettleGonePairTeardown`, the `--reset` scenario, when the `--reset` refusal happens relative to network access in each clone form, and the `## Merge danger` door and blast radius against every outward push and changed caller."
---
