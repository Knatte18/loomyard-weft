# Card 5 — Reed names a strand pane found in another session

**Edit:**
- `internal/reedengine#StrandStatus`
- `internal/reedengine#Engine.Status`
- `internal/reedengine#planReconcile` -> `func planReconcile(strands []Strand, live []LivePane, windows []windowPane, policy reapPolicy, elsewhere map[string]string) reconcilePlan`
- `internal/reedengine#Engine.reconcileLocked`
- `internal/reedengine#TestPlanReconcile`
- `internal/reedengine#TestStatus_ReportsOwnWindowStrandsFromTheSessionListing`
- `internal/reedengine#callVerb`
- `internal/reedengine/doc.go`

**Create:**
- `plan:internal/reedengine#serverListVerb` -> `const serverListVerb = "list-panes-all"`
- `plan:internal/reedengine#panePlacementFormat` -> `const panePlacementFormat = "#{pane_id} #{session_name} #{window_index}"`
- `plan:internal/reedengine#TmuxCmd.panePlacements` -> `func (p TmuxCmd) panePlacements() (map[string]panePlacement, error)`
- `plan:internal/reedengine#panePlacement` -> `type panePlacement struct`
- `plan:internal/reedengine#Engine.strandsElsewhereLocked` -> `func (e *Engine) strandsElsewhereLocked(strands []Strand, windows []windowPane) map[string]string`
- `plan:internal/reedengine#TestReconcileLocked_KeepsABindingFoundInAnotherSession` -> `func TestReconcileLocked_KeepsABindingFoundInAnotherSession(t *testing.T)`

**Uses:**
- `internal/reedengine#TmuxCmd.listWindowPanes`
- `internal/reedengine#fakeTmux`
- `internal/reedengine#newTestEngine`
- `internal/reedengine#TestRemoveStrand_NeverKillsAPaneOutsideThisSession`

**Intent:**
D2: a strand's recorded pane that the reed session's session-wide listing lacks is looked up once across the whole server; found in another session, the strand is dead (it already is: `presentPanes` never holds it), its status row names the placement, and reconcile keeps its binding instead of clearing it.

`panePlacements` runs `list-panes -a -F panePlacementFormat` on reed's `-L` server and maps each pane id to a `panePlacement` holding its session name and window index, skipping a line with fewer than three fields; `panePlacement` has a `String()` method answering `session:window`.
`Engine.strandsElsewhereLocked` collects the bound pane ids of own-window strands (`isOwnWindow`, which every strand that owns a pane is) that `windows` does not list; with none it returns nil and runs no tmux command, which is the bound "the lookup runs only when a strand's pane is absent from the session".
Otherwise it calls `panePlacements` once and returns, per such pane id, the placement of a pane listed in a session other than `e.SessionName()`.
A failed lookup is logged at Warn and returns nil, so the binding is cleared as today and `Status` does not fail; a pane in another tmux server is not listed and is cleared as today.

`StrandStatus` gains `Elsewhere string`, the `session:window` placement, empty when none.
`Engine.Status` fills it from `strandsElsewhereLocked(st.Strands, windows)`.
`planReconcile` takes the map: an own-window strand whose pane `windows` does not list keeps its binding when the pane id is in `elsewhere`, and is cleared otherwise, as today; nothing in `elsewhere` is ever killed, since the kill sets read `live` alone.
`Engine.reconcileLocked` computes the map with `strandsElsewhereLocked(st.Strands, windows)` and passes it, so every public reed op's reconcile keeps the placement until the pane leaves the server or a removal drops the binding.
Removal is unchanged: `killStrandPanes` already narrows its kills to panes the session lists (`paneIDsInSession`), so it drops such a strand's binding and never kills its pane, which `TestRemoveStrand_NeverKillsAPaneOutsideThisSession` already pins.

The test fake: `callVerb` names a `list-panes` call carrying `-a` `serverListVerb`, so a test scripts the server listing apart from the session listings; unscripted it answers empty, which `strandsElsewhereLocked` reads as found nowhere.

`internal/reedengine/doc.go` gains the placement rule: the server-wide lookup, its bound, the kept binding and its end, and that a pane id reused after a server restart can name a sibling worktree's pane until the binding goes (D2's Bound).

Tests:
- `TestPlanReconcile` passes nil `elsewhere` in its existing rows and gains a row where an own-window strand's pane is absent from `windows` but in `elsewhere`: no cleared guid, nothing killed.
- `TestStatus_ReportsOwnWindowStrandsFromTheSessionListing` scripts `serverListVerb` with `%9 other 2` for its existing "gone" strand and gains a strand on `%8`, absent from every listing: "gone" reads `Elsewhere: "other:2"`, not live and not present, and the new strand reads `Elsewhere: ""`; it also asserts one `serverListVerb` call.
  A further row scripts `serverListVerb` to fail: `Status` still succeeds and "gone" reads `Elsewhere: ""`.
- `TestReconcileLocked_KeepsABindingFoundInAnotherSession` is new, since the reconcileLocked tests drive window-0 strands only: with the fake scripting the session listing without the strand's pane and `serverListVerb` placing it in session `other`, `reconcileLocked` keeps the own-window strand's binding and kills nothing; with `serverListVerb` scripted to fail, it clears the binding.

**ImpactSummary:** One new tmux read on the path where a bound pane is missing, a new field on `StrandStatus` that card 13 reads, and `planReconcile` re-signed with its one production caller on this card; no kill or removal path changes.

**Commit:** 5: reed names a strand pane found in another session and keeps its binding
