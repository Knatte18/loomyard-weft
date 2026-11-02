---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/reedengine/buttonclick.go:101
    summary: the existence check runs on the rendered spec.Dir only, so a recorded anchor subpath missing on disk reaches the run or the prompt unchecked
    fix: In runButtonVia, after anchorOf returns runDir, stat runDir as the existing check stats spec.Dir, and when it is missing say so (naming runDir, and that the anchor directory is gone) and return before the confirm prompt and the run; add a TestRunButtonVia row for a runDir that does not exist.
  - id: F2
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/reedengine/buttonclick_test.go:84
    summary: no row combines an unresolvable executable with a moved anchor, so the prompt-failure message naming runDir/.lym/logs could regress to spec.Dir unnoticed
    fix: In TestRunButtonVia, add a row with exeErr set, anchorMoved true and a confirming entry, wanting the message that names the log directory under the moved anchor (backend subdirectory), or set anchorMoved on the existing exeErr row.
  - id: F3
    severity: NIT
    class: consistency
    origin: lens:code-correctness
    location: internal/reedengine/buttonclick.go:113
    summary: the runDir != spec.Dir Info log also fires when the paths differ only by a symlink, since ResolveWorktree resolves symlinks, so its stated reason can be false
    fix: Compare with symlinks resolved on both sides, or reword the Info message to say the run directory differs from the rendered one without claiming the anchor changed.
---

### [LOW] Missing anchor directory is never checked

**Location:** `internal/reedengine/buttonclick.go:101`
**Issue:** The existence check stats `spec.Dir`, and `runDir` from `anchorOf` is used unchecked. `readRecordedAnchor` validates only the marker text, so an anchor naming a subpath absent on disk gives a `runDir` that does not exist. A plain entry fails to start and shows a chdir error; a confirm entry raises its prompt, the confirmed line's directory change fails with its output discarded, and the confirmed pass then fails the same way after the operator answered y. The fasit's expected outcome is a click that works, or a clear message to reattach; this path yields neither. The case is narrow, a marker set before its directory exists.

### [LOW] Prompt-failure message not pinned to the moved anchor

**Location:** `internal/reedengine/buttonclick_test.go:84`
**Issue:** The message `could not raise its prompt; see the lym log in <runDir>/.lym/logs` was changed from `spec.Dir` to `runDir`. The only row that exercises it leaves the anchor unmoved, where `runDir == dir`, so reverting that line passes every row.

### [NIT] Info log can claim an anchor change that is a symlink difference

**Location:** `internal/reedengine/buttonclick.go:113`
**Issue:** `ResolveWorktree` calls `EvalSymlinks`, so under a symlinked hub path `runDir` differs from `spec.Dir` on every click and the log says the rendered directory is not the anchor. Behaviour is correct, since the verb's gate compares with `samePath`; the log is misleading.

## Rejected

- Lens code-correctness, buttons rendered for a `window`-verb directory outside any hub resolving to the repo root: unconfirmed by the lens itself; the button spec's `Dir` is rendered from the hub geometry's anchors (`internal/hubgeom/buttons.go`), always inside a hub worktree.
- Second candidate cause (task session in the parent's or prime's anchor) left unaddressed: the decision record states no code path producing it was found, and the diff's afresh anchor read covers whichever worktree holds the rendered directory, so it is no gap.
- `RunButton` nil `anchorOf` reaching a click: the only production caller passes a non-nil function.
- Unasserted Info and Warn log lines: the repository's other log lines in this file are not asserted either.

## Fasit check

The diff makes the change the entry's `## Fix` names: the click reads the worktree's anchor afresh and runs there, and a test sets the anchor marker after the render and then clicks (`TestButtonRunDirectory` rows, `TestRunButtonVia` `AnchorMoved…` rows). `ResolveWorktree(spec.Dir).AnchorPath()` is used in place of `NamedWorktree(hub, name)`, which needs no hub and reads the same marker. `PATTERN-cwd-resolution` holds: `reedengine` is handed the reader and imports no `lymcwd`. The commit has no `Pattern-Departure:` trailer and none is needed. No `_lym/landing/summary.md` claim was checked, as none was read.
