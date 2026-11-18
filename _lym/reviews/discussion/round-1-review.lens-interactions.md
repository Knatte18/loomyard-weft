```yaml
lens: interactions
outcome: reported
```

Task: interactions lens. I checked each pair of decisions that act on the same state.

**Pairs checked**
- D1 × D2, the builder call: D1 asks for beside-runs, and the builder reads the recipe from the seed. **Held.**
- D1 × D2, the shared sidecar path: D1 writes the sidecar with the current build, and the bootstrap's `resolveStatusStrandAction` then keeps the strand. **Held.**
- D1 × the bootstrap's `ensureStatusStrand`, the race where the bootstrap adds first: D1's role-held refusal counts as done. **Held.**
- D1 × the bootstrap's `ensureStatusStrand`, the race where D1 adds first: the bootstrap's own add is refused and `lym loom start` fails. **Broken** (F1).
- D1, the runs sidecar write × the re-add order, failure path: **Broken** (F2).
- D2's group in reed state × D3's window read from reed state: **Broken** (F3).
- D3's reed state read × the Constraint "takes no lock": **Broken** (F4).
- D1's not-done test × D3's not-done test: both use `shedengine.StateDone`, and both treat an absent run the same. **Held.**
- D1 × the Constraint "never fails `lym beam start`": every failure is a warning. **Held.**
- D2's overview placement × reed's refusal of a group on an overview strand × the D2 criterion "overview gives no group": **Held.**
- The D1 criteria × D1's failure and role-held paths: **Held**, except that no criterion covers a failed runs sidecar write (F2).
- The D3 criteria × D2's `window_share_max` 0 case: no criterion pins a window name for that case. Covered by F3.

**F1 — MEDIUM, design.** Location: `decision-record.md` D1, role-held refusal, and Open risks bullet 2.
D1's new add of the loom status strand can win a race against a concurrent hand `lym loom start` or resume.
The bootstrap's `AddStrand` is then refused because the role is held, and `ensureStatusStrand` returns that error.
`lym loom start` then fails before it spawns a driver.
D1 treats the refusal as done only on its own side.
The Open risk says "D1 does not widen" the race, which is false: before D1, an add after the runs strand's replace succeeded as an own-window strand.
Evidence:
- `internal/loomcli/sharedbootstrap.go:312-314`: `AddStrand` error returned.
- `internal/loomcli/start.go:553-556`: the error fails the start.
- `internal/reedengine/strand.go:224-228`: the role-held refusal.

**F2 — MEDIUM, design.** Location: `decision-record.md` D1, "writes the runs sidecar as today. Then…".
Today a failed runs sidecar write returns early with a warning naming `lym beam start`.
D1 does not say whether the re-add still runs after that failure.
If the code returns early as it does today, the checklist is gone.
The next `lym beam start` then takes the `runs != nil` branch and never re-adds it, since D1 triggers only on the loom status branch.
So the warning's way forward does not bring the checklist back.
Evidence: `internal/orchcli/runs.go:84-101`, where the runs sidecar failure returns early and the `runs != nil` replace path is separate.

**F3 — MEDIUM, design.** Location: `decision-record.md` D3, "the window the loom status strand holds, read from the session's reed state", and its Rationale.
D2 writes the recipe as the strand's group (`Display.Window`) into reed state.
Reed state records no window name.
When `window_share_max` is 0, or the group's record is not shared, the window is named by the role, `loom-status`, while reed state still reads `pawl`.
A reader taking the group from reed state names the wrong window, which is the very case D3's Rationale says it handles.
Getting it right needs the group's `WindowGroups[...].Shared` record, which reed state does hold, or a tmux read; D3 names neither.
Evidence:
- `internal/reedengine/state.go:21-53`: `Strand` has no window name.
- `internal/reedengine/spawn.go:185-186,197`: naming apart versus shared.
- `internal/reedengine/ownwindow.go:204-224`: placement planning.

**F4 — LOW, consistency.** Location: `decision-record.md` Constraints, "The overview reads files only, takes no lock", against D3's read of reed state.
The reed state reader, `reedengine.LoadState`, goes through `state.ReadJSON`, which takes a shared read lock on `reed.json.lock`.
So D3's read contradicts the Constraint unless the record names a lock-free reader.
Evidence:
- `internal/reedengine/state.go:150-157`.
- `internal/state/state.go:96-107`.

**Tool calls**
- Read `internal/loomcli/start.go:540-569`: to see how `lym loom start` handles an `ensureStatusStrand` error. My context had only the call site line from a grep.
