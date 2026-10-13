---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: _lyx/plan/05-window-order-and-shell.md (Intent, the new-file paragraph and the shellWindowOf paragraph)
    summary: Card 5 puts shellWindowOf, which reads st.SelvagePaneID, in the new file windoworder.go, which TestSelvageIdentifiersConfinedToSelvagePane forbids, so go test ./... fails
    fix: In card 5's Intent, change the sentence placing the new members so that shellWindowOf lives in internal/reedengine/selvagepane.go beside recordedPaneIDs (the file selvagePaneAllowlist admits for every Selvage-naming identifier), and add that no other new member in windoworder.go names a Selvage identifier outside a call position; the Create glyph plan:internal/reedengine#shellWindowOf stays as it is, since it is package-scoped, so no target changes.
  - id: F2
    severity: LOW
    class: scope
    location: _lyx/plan/07-focus-rules.md (Intent, the doc.go paragraph)
    summary: Three internal/reedengine/doc.go sentences that cards 6 and 7 falsify are named for no rewrite - the own-window option set, the multi-client warning never changing the argv, and the current window being the strands' window
    fix: "In card 7's Intent, after the two doc.go sentences it already names, add three more for rewrite: the Own windows sentence 'The window carries @lyx_own, @strand_color and @lyx_unseen, each set by window id' (an overview window sets neither @strand_color nor @lyx_unseen, and a new agent window the focus rule selects gets no @lyx_unseen); the multi-client warning bullet's 'It never blocks, never changes the argv' (its other-client report now drops the attach selection, and a failed listing counts as another client); and the Detached windows sentence 'one chained after a detached new-window without a target lands on the current window, the strands' window' (the current window is no longer assumed to be the strands' window, since attach and the focus rules select agent and overview windows)."
  - id: F3
    severity: NIT
    class: consistency
    location: _lyx/plan/07-focus-rules.md (Intent, the TestAttachArgv_MultiClientWarning sentence)
    summary: Card 7 makes the ListClientsError_WarnsAndDoesNotChangeBehaviour subtest assert a behavior change (no select-window appended) but keeps its name, which now says the opposite
    fix: In card 7's Intent, add that the subtest ListClientsError_WarnsAndDoesNotChangeBehaviour in internal/reedengine/attach_test.go is renamed ListClientsError_WarnsAndAppendsNoSelection, alongside the fixture and assertion changes the card already lists.
---

### [BLOCKING] Card 5 places a Selvage-identifier read outside the Selvage seam

**Location:** `_lyx/plan/05-window-order-and-shell.md`, Intent: "The new members live in a new file, `internal/reedengine/windoworder.go`, package `reedengine`, except `newWindowLocked`" and "`shellWindowOf` returns the window of `st.SelvagePaneID`".

**Issue:** `internal/reedengine/selvagepane_enforcement_test.go`'s `TestSelvageIdentifiersConfinedToSelvagePane` parses every non-test file of `internal/reedengine` and fails on any Selvage-naming identifier outside a call position or composite-literal key, in any file but the three `selvagePaneAllowlist` names (`selvagepane.go`, `state.go`, `config.go`).
Its own fixture test, `TestSelvageIdentViolations_ExemptionsAndFlags`, pins that an `st.SelvagePaneID` selector is a violation.
The existing code honours this: `strandWindowTarget` in `windowtarget.go` reaches the Selvage pane only through `recordedPaneIDs`, which lives in `selvagepane.go`.
Card 5 puts `shellWindowOf`, whose whole job is reading `st.SelvagePaneID`, in the new `windoworder.go`, which is on no allowlist, so the plan's `go test ./...` fails on the card that lands it.
This is the same defect class as the round's first focus item, the `new-window` literal and `windowCreationAllowlist`, on a second scan: card 5 handles that allowlist, but not this one.
Moving `shellWindowOf` into `selvagepane.go` is the fix that matches the seam.
Adding `windoworder.go` to `selvagePaneAllowlist` would weaken the seam and would also change a target list, so the fix does not propose it.

### [LOW] doc.go sentences falsified by cards 6 and 7 are not named for a rewrite

**Location:** `_lyx/plan/07-focus-rules.md`, Intent, the paragraph naming the `doc.go` rewrites; the sentences are in `internal/reedengine/doc.go` (Own windows paragraph, Detached windows paragraph, and the multi-client warning bullet).

**Issue:** I checked each focus-category sentence of `internal/reedengine/doc.go`.
The `OpenWindow` argv sentence and the `@lyx_hidden` line are named in card 5.
The own-window `new-window -d, so it never takes focus` sentence and the removal sentence are named in card 7.
The geometry-pins `"status-format[0]" (the VIEW, strand and window buttons)` bullet is named in card 8.
Those are covered.
Three neighbouring sentences go false and no card names them:
"The window carries @lyx_own, @strand_color and @lyx_unseen, each set by window id" is false for an overview window (card 6 sets neither colour nor unseen) and for a new agent window the focus rule selects (card 7 sets no `@lyx_unseen`).
The multi-client warning bullet says the warning "never blocks, never changes the argv", but card 7 makes `warnMismatchedClientsLocked`'s report drop the attach selection.
The Detached windows sentence calls the current window "the strands' window", which attach and the focus rules no longer make true.
Card 7's general "doc.go say so" covers the focus bound, not these specific claims.
The Constraints require `internal/reedengine/doc.go` to land in the same commit.

### [NIT] A subtest name contradicts its new assertion

**Location:** `_lyx/plan/07-focus-rules.md`, Intent: "its "ListClientsError" row ... asserts no `select-window` is appended".

**Issue:** The subtest in `internal/reedengine/attach_test.go` is named `ListClientsError_WarnsAndDoesNotChangeBehaviour`.
After card 7, a failed client listing does change behaviour: it drops the attach selection, and the row asserts exactly that.
The name now misstates what the row pins.

## Focus items settled

- `new-window` in `window.go` against `windowCreationAllowlist`: card 5 lists `internal/reedengine#windowCreationAllowlist`, drops the `window.go` entry, rewrites the `spawn.go` entry's `Why`, and keeps the literal in `spawn.go`'s `newWindowLocked`.
  Card 6's `overviewWindowInsertArgs` and card 8's `next-window`/`previous-window` bindings move no allowlisted literal.
  `probe.go`'s entry stays valid.
  Not a defect; the Selvage seam scan is F1.
- doc.go sentences: see F2 for the sentences that are not covered.
- Test economy of the pure placement rows: `TestWindowOrder`'s no-shell and no-window rows, and the hidden-lowest and no-window rows for `overviewWindowInsertArgs`, are states a real session cannot reach (`Up` always creates Selvage, and a real session always has a window, with hidden windows after the shell).
  The plain-placement cases are left to `TestWindowOrder_RealTmux` alone.
  No pure row pins a placement a real-tmux row pins; not a defect.
- Card 5's `OpenWindow` under a nil `LoadState` result: the card says a nil state records no Selvage pane and the check does nothing, logged at Debug, which matches `LoadState` returning `nil, nil` with no `reed.json`.
  It states `TestOpenWindow`'s exact `wantArgv` as `new-window -d`, then the placement (`-t` on the exact session target when `list-windows` is unscripted), then the caller's arguments.
  Both are stated; not a defect.
- Card 6's window-group refusal: `validateDisplay` in `strand.go` is the first display check and calls `render.ValidateDisplay` only from its own switch.
  So a direct call reaches both clauses, and card 6's `validateDisplay` subtest has a row for `Window` non-empty and a row for `Window` empty with `WindowSize` non-zero.
  Each clause is reached; not a defect.
