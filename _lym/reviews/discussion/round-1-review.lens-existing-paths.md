```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced every existing code path that writes or reads the state D1 to D4 change: the loom-status strand and its placement, the loom status sidecar, reed window names, the viewer and the overview frame.

**Paths traced, per decision**

D1, the shared beside spec and sidecar path:
- Loom start's keep, replace or add (`internal/loomcli/sharedbootstrap.go:281-322`, `ensureStatusStrand`): **survives**. It builds from the shared `statusStrandAddSpec`. Beside a runs strand a replace keeps the own window and Title, and a matching sidecar keeps the strand.
- Orch's build identity (`internal/orchcli/cli.go:241` sets `c.currentBuild = statuspane.CurrentBuild`): **survives**. It is the same identity loom's `resolveStatusStrandAction` compares, so a sidecar orch writes reads as current to loom start.
- A pawl driver of an older build running the old `ensureRunsStrand` in-process at Beam-Up: **survives**. Scope Out covers it.

D2, the move and its failures:
- Reed's replace (`internal/reedengine/replace.go:64-99`, `ReplaceStrand`): **survives** for the move itself. It removes before the add, which is why the record has a failure criterion for each side of the removal.
- Reed's viewer handling on that replace (`replace.go:70-71,97-98`; `strand.go:835-857`; `ownwindow.go:456-471`; `spawn.go:256`): **breaks**. This contradicts D2's viewer claim; see finding 1.
- `lym beam start` after a reed server restart (`lifecycle.go:3-5,743-746`): **survives**. Up launches no strand, so both strands read dead. Runs is replaced as not current and loom-status reaches D2 case 2.
- `lym reed resume`, the only replayer (`lifecycle.go:5`), reaching `createOwnWindowPaneLocked` and then `openOwnWindowLocked`: **survives**. It relaunches loom-status from its persisted beside display, and under D3 names the window by Title the same way.
- The sidecar after the failed-replace fallback: **survives**. A stale GUID makes loom start add again.

D3, the window name:
- A new own window (`spawn.go:165-167`): **survives**, covered by "the new window".
- A grouped strand placed apart (`spawn.go:185-186`, `placeApart`): **ambiguous**; see finding 2.
- A group's shared window (`spawn.go:197`, named by the group): **survives**. D3 says a group ignores Title.
- Break-out from the shell window (`ownwindow.go:405`): **survives**, covered.
- The OpenWindow name lookup (`window.go`, doc.go:385-386): **survives**. It skips windows carrying `@lym_own`, so a Title-named own window is never adopted or killed by it.
- Shell-window repair (`windoworder.go:204-206`): **survives**. It finds the shell window by console pane, and D3 reserves the name `shell`.
- Pane-title repair (`namerepair.go:61-76`): **survives**. It touches pane titles, not window names.
- Existing writers of `Display.Title`: **survives**. Only `orchcli/runs.go:44` sets it, on an overview display, so no persisted own-window strand changes name.

D4, the overview line:
- `overviewSource.frame` and the `DobbyRunCells` readers (`internal/battencli/overview.go:260-300,449-460`): **survives**. `ReadSeed` and the status cache poll are lock-free readers, and the frame already composes line groups.

**Findings**

1. **BLOCKING · design · `_lym/discussion/decision-record.md` D2 ("The viewer ends on the beam overview window, as after today's replace") and its criterion "After the move the viewer is on the beam overview window".** Reed's existing paths do not bring the viewer to the beam overview after the move, and no Decision adds a step that does.
   - Replacing the overview `loom-status` with an own-window spec calls `killStrandPanes(..., selectOnEnd=true)`, because the replacement is not an overview (`replace.go:70-71`). A viewer on the ending window is moved to `relevantWindow`: the newest live agent window, else the overview window. At that moment no overview window exists, so it lands on an agent window or is left to tmux (`strand.go:835,855-856`; `ownwindow.go:456-470`).
   - The new `KeepViewer` window is never selected (`ownwindow.go:479-480`).
   - The following `AddStrand` of the runs overview never selects its window (`spawn.go:256`: "An overview window is never selected here").
   - Today's replace reaches the overview only through `endedCurrent && replacingWithOverview` (`replace.go:97-98`), which the move no longer meets.
   - "as after today's replace" is therefore a false premise, and the criterion cannot pass without a mechanism the record does not design, such as a reed select of the overview window.

2. **LOW · consistency · `_lym/discussion/decision-record.md` D3 ("an own-window strand placed apart" and "A strand in a shared window group ignores `Title`").** Reed's existing "placed apart" path is a *grouped* strand's own window: `placeApart` at `internal/reedengine/spawn.go:185-186` opens it with `strandLabel(s.Name)`. D3's two sentences disagree on whether such a strand takes `Title`, and the D3 criterion's "placed apart" wording inherits the ambiguity.

Tool calls:
1. Read the `openOwnWindowLocked` callers, the rename sites, loom's `ensureStatusStrand` and `resolveSeedRecipe`: the context had reed's naming function but not which paths call it, nor loom's sidecar consumer.
2. Read the `spawn.go` launch branches, orch's `reedUp` and `currentBuild` seam, and the reed Up/Resume docs: needed to check the restart and resume paths and the build identity the sidecar compares.
3. Read `reedUp` and the Resume and Up semantics: needed to settle whether `lym beam start` relaunches dead strands before `ensureRunsStrand` runs.
