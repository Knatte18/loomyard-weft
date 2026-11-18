```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: each claim and bound in the record was tested against the code, looking for a concrete counterexample.

**Claims attacked**
- C2, second overview strand refused (`internal/reedengine/strand.go:404-410`): held.
- C2, window group refused on an overview strand (`strand.go:75-76`): held.
- C2 and D1, role-held refusal: held only in part. A non-live holder is first reclaimed (`strand.go:663`), and the refusal is an untyped `fmt.Errorf` (`strand.go:226`). See F3.
- C3, `ReplaceStrand` removes the old strand and adds the new spec whatever its anchor: held, with exceptions the record does not need. It refuses a parent strand, and an overview spec while a third strand holds the overview. The removal is persisted even when the add fails (`replace.go:102-115`), which F1 relies on.
- C4, `window_share_max` 0 places a size-1 group apart, named by role (`ownwindow.go:215-219`, `spawn.go:185-186`): held.
- C5, `KeepViewer` never selects the window, and an overview window is never selected (`ownwindow.go:479`, `spawn.go:286-291`): held.
- C6, done is exactly `shedengine.StateDone` (`anchordeps.go:420`): held.
- C8, the overview reads files only and takes no lock: broken for the reed-state read. See F4.
- D1, the seed, status and sidecar are the session's own worktree, never the caller's: held. Every `BeamUp` caller (`loomcli/pawlwiring.go:107`, `beamcli/cli.go:248`, `adoptcli/deps.go:133`, and `BeamUpWorktree` through `bringUpLocation`) passes the beam's own location. That becomes `c.location` and the reed geometry (`orchcli/cli.go:203-209`). The record names neither `wireCLI` nor `c.location` as its evidence.
- D1 Rationale, `statusStrandAddSpec` already gives the checklist its own window when the runs strand came first (`loomcli/bootstrap.go:33-37`): held.
- D1 Bound, it adds only right after it took the strand's window: held as written, but it leaves two loss paths open. See F1 and F2.
- D1, a role-held refusal counts as done because a concurrent `lym loom start` added it: broken. See F3.
- D3 Rationale, reading the window reed holds stays right when `window_share_max` is 0: broken. See F5.
- D3, seed and status reads take no lock (`shedrun/seed.go:113`, `statuspane/cache.go:64`): held.
- Open risk, a hand `lym loom start` racing `lym beam start`: held. It predates the task, and its existing failure is a refused overview add.

**Findings**

**F1 — MEDIUM, design. `decision-record.md` D1, Bound line.**
A failed add inside `ReplaceStrand` loses the checklist for good.
Counterexample:
1. `ReplaceStrand(loomStatus.GUID, runsSpec)` removes the status strand, kills its pane and persists the removal.
2. The add then fails, for example on a launch failure (`replace.go:65`, `80-85`, `104`, `111-112`).
3. `ensureRunsStrand` returns the warning naming `lym beam start`.
4. Rerunning `lym beam start` finds neither a runs strand nor a status strand, takes the `default:` `AddStrand` branch (`orchcli/runs.go:92-95`), and D1 never re-adds the checklist there.

The checklist stays gone while pawl lives, and the warning names the wrong verb for bringing it back. D1 should state this path: either the warning also names `lym loom start`, or the re-add runs whenever the self run is not done and no status strand remains.

**F2 — LOW, design. `decision-record.md` D1, "writes the runs sidecar as today. Then…".**
D1 does not say whether the re-add still runs when the runs sidecar write fails. Today that failure returns early (`runs.go:98-100`).
Counterexample, if the early return stays:
1. The runs strand took the window and the sidecar write failed, so the checklist is not re-added.
2. The next `lym beam start` finds a runs strand that is not current (it has no sidecar) and takes the `case runs != nil` replace (`runs.go:84-87`), which never re-adds.

The checklist is lost. D1 should order the re-add before the early return, or say that it runs regardless of that write.

**F3 — MEDIUM, design. `decision-record.md` D1, "An add refused because a strand already holds the loom status role counts as done", and Constraints bullet 2.**
Reed reports the role-held refusal as an untyped `fmt.Errorf` (`strand.go:226`). Its only sentinels are `ErrNoSession`, `ErrUnknownStrand` and the window errors. Telling this refusal apart from any other failure would mean matching the message text, a fact the record leaves for Plan to find.
The claim also overreaches. A dead holder is reclaimed, not refused (`strand.go:663`). A holder kept because the pair is paused, or because it is hidden, is refused and counted as done although no checklist shows.
Reed already offers a race-free route: `AddStrandUnless(spec, unlessName)` returns `skipped=true` when the name is held (`strand.go:550-604`). However, orch's `strandOps` seam exposes only `Strands`, `RemoveStrand`, `AddStrand` and `ReplaceStrand` (`orchcli/cli.go:30-35`).
D1 should state how the refusal is recognised, for example through `AddStrandUnless` with the seam widened, and pin it.

**F4 — MEDIUM, consistency. `decision-record.md` Constraints, "The overview reads files only, takes no lock", against D3's read of the session's reed state.**
The one reader of reed's state, `reedengine.LoadState`, reads through `state.ReadJSON`, which takes a shared lock on `reed.json.lock` (`reedengine/state.go:150-154`, `state/state.go:96-102`).
D3's window read either breaks the Constraint, or needs an unlocked reader the record does not name.

**F5 — BLOCKING, design. `decision-record.md` D3, "The line names the window the loom status strand holds, read from the session's reed state", and its Rationale.**
`reed.json` records no window name. A `Strand` holds only `Display.Window`, the group, and `PaneID` (`reedengine/state.go:21-53`). The window's name lives in tmux alone.
At `window_share_max` 0 the strand's `Display.Window` is still the recipe, `pawl`, but reed opens the window apart under the role `loom-status` (`spawn.go:185-186`, `ownwindow.go:215-219`). Reading "the window reed holds" from its state therefore names `pawl`, which is wrong in exactly the case the Rationale says the read handles.
Getting it right means either recomputing the name from `WindowGroups[group].Shared` together with the role, which is the recomputation the Rationale rejects, or querying tmux. The record names neither. That is a fact the design rests on, left for Plan.

**Tool calls**
1. `sed -n 580,700p internal/reedengine/strand.go`: the `AddStrandUnless` body and where `reclaimDeadNameLocked` runs before the role-held refusal. My context had only their signatures and doc lines.
2. `grep` of `shedrun.ReadSeed`, orch's `strandOps` and `StatusCache.Poll`: whether the seed and status reads lock, and which reed operations orch's seam exposes. None of these bodies had been read.
