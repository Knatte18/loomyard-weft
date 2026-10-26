```yaml
lens: fix-claims
outcome: reported
```

fix-claims: I checked whether the code at HEAD supports each claim the fix added or rewrote. One claim overstates it.

1. **MEDIUM**: D5's upgrade bound overstates what an older build fails to write.
   - Changed text: "so once a beam project carries `recipe: pawl` that build's board replay refuses the entry and its writes to the board fail until the machine is updated".
   - Summary: an older build fails only its writes to that one entry, not every write to the board.
   - Evidence: `normalizeTask` has two callers outside tests. One is upsert (`internal/boardengine/task.go:388` and `:445`), which checks only the entry being written. The other is replay (`internal/boardengine/replay.go:309` and `:391`), which calls `r.refuse(...)` for that entry's change alone and moves on. Load does not call `normalizeTask`, so an older build still loads the board and writes its other entries. The reviewer's fix said "its writes to it fail", meaning that entry. The fixer turned this into "its writes to the board fail".

The other new claims hold at HEAD:
- The `respawnCap` cap and the driver-dead notice past it (`innerrun.go:186,445`, `notice.go:384-385`).
- The `checkProjectFields` recipe refusal (`task.go:289,307`).
- The `beamLandRefusal` orch and watcher cases (`beamland.go:86-92`).
- The entry merge path, which exists as `MergeTasks` (`board.go:208`, `store.go:998`).

Tool calls:
- Grepped `replay.go`, `notice.go` and the merge functions in `boardengine` to check the replay, notice and merge claims. My context did not have those files.
- Read the `replay.go` lines 300-320 and 385-398 to see how replay handles a `normalizeTask` failure. My context did not have that code.
- Grepped every `normalizeTask` caller to learn whether load normalizes every entry. My context did not have the caller list.
