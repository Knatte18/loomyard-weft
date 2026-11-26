```yaml
lens: fix-code-correctness
outcome: reported
```

Task: fix-code-correctness check of this round's change set (`primeOfBoard`, `readBeamMode`, `modeHeader` and the two tests).

**Findings**

1. **LOW.** Changed text: the `readBeamMode` absent-worktree error (`internal/hubgeom/beammodes.go`), against the unchanged legend line "A setting that the root and every beam take from the hub-wide default is stated here alone." in `modeHeader` and its twin sentence in `internal/boardengine/doc.go`.
   - **Summary:** an absent beam now drops out of `everyPlaceHubWide`, so the legend can claim something about "every beam" that the render did not check.
   - **Input:** the root and every present beam take `autonomy` from the hub-wide default (`off`), and beam X, whose worktree is on another machine, sets `autonomy: true`.
   - **Trace:** `BeamModes` leaves X out of `Beams`. `everyPlaceHubWide` ranges over `b.modes.Beams` only and returns true. Autonomy is left off every mode line, and X's section shows no line.
   - **Result:** the README states a hub-wide `off` with "A setting that the root and every beam take from the hub-wide default is stated here alone", so a reader takes X to be off. Nothing in the README says a section with no mode line is unread.
   - **Evidence:** before the fix, only a failed read left a beam out. It is now the normal case on a hub that spans machines.

I found no other defect:
- **`readBeamMode`:** an absent worktree and a failing stat both return an error. Its only caller, `BeamModes`, warns and skips the beam.
- **`primeOfBoard`:** a listing failure is returned wrapped. The main worktree comes first in both the dotgit and porcelain listings. `lymcwd.ResolveWithin` goes through `dotgit.FindRoot`, and the `ErrRefStorage` refusal I found is in `ListWorktrees`, not there.
- **`TestBeamModes`:** with the defect in place, `bm-elsewhere` has no origin record and would be returned in `Beams`, so `cmp.Diff` fails.
- **`TestRenderToDisk_ShowsTheInstalledBeamModes`:** if a failed read still returned modes, the header would render and the new row would fail. A broken open-beam filter would fail the `gotBeams` comparison.

**Further tool calls**
- One `grep` for `gitWorktreeRoot` and `ErrRefStorage`: my context did not show whether `lymcwd.ResolveWithin` also refuses a repository whose refs are not files.
