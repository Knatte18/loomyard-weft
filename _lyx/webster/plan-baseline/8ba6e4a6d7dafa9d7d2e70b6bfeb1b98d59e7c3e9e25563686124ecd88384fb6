# Card 6 — A reed add reclaims a dead strand's name

**Edit:**
- `internal/reedengine#Engine.AddStrandUnless`
- `internal/reedengine/doc.go`

**Create:**
- `internal/reedengine#Engine.reclaimDeadNameLocked`
- `internal/reedengine#TestAddStrand_ReclaimsADeadHoldersName`

**Uses:**
- `internal/reedengine#Engine.strandNameLocked`
- `internal/reedengine#Engine.removeStrandLocked`
- `internal/reedengine#Engine.killStrandPanes`
- `internal/reedengine#reapPaneChildren`
- `internal/reedengine#Engine.pairPaused`
- `internal/reedengine#Engine.listStrandAndWindowPanes`
- `internal/reedengine#presentPanes`
- `internal/reedengine#Engine.engineAliveIDSet`
- `internal/reedengine#TestAddStrandUnless_NoNamedStrandAdds`
- `internal/reedengine#TestStrandNameLocked_ExplicitNameHeldRefuses`
- `internal/reedengine#fakeTmux`
- `internal/reedengine#newTestEngine`

**Intent:**
D6: before `AddStrand` forms a name, reed removes a strand holding the name the add asks for when that holder is not live per D1, not hidden, and the pair is not paused; numbering then runs over what remains.
This fixes #799, #802 and the name clash of #811, where a dead registration held the canonical name, so a respawn was numbered or refused by seatengine's `checkName`.

`Engine.reclaimDeadNameLocked` runs in `AddStrandUnless` on the ordinary add path, right before `addStrandLocked`: after the `unlessName` skip and after the `--if-absent` branches, so an `--if-absent` match still relaunches its strand and `AddStrandUnless` still returns a named strand untouched.
The asked-for name is `explicitName` when `spec.NameOverride` is set, else the un-numbered role name `agentname.Format(shortname, slug, role)` for `spec.Role` (or `defaultRole`).
The holder is the first strand of `st.Strands` with that exact name; a numbered form is never reclaimed (D6's Bound).
The holder keeps the name, and the method returns nil having changed nothing, when it is hidden (`anchor:hidden`), when `e.pairPaused()` (Resume planning holds a paused pair's dormant strands on purpose), when it has children (the ordinary removal refuses a non-leaf without recursion, and reclaim never cascades), or when it is live by `engineAliveIDSet(presentPanes(...))` over a fresh `listStrandAndWindowPanes` listing, which includes a holder inside D1's launch grace.
A failed listing is returned as the add's error, since the add then lists panes again to launch and would fail the same way.
Otherwise it removes the holder with reed's ordinary removal: `removeStrandLocked(st, guid, false)`, then `killStrandPanes(st, paneIDs, false)`, which kills only a pane the session lists and so never a pane D2 places elsewhere, then `reapPaneChildren` on the returned pids with `reapExitTimeout`, and logs `reed: reclaimed a dead strand's name` at Info with the name and the old guid.
The add's own save then persists the removal together with the new strand; a launch failure after the reclaim returns before any save, so reed.json keeps the old holder and only its dead pane is gone, which the next reconcile clears.
`strandNameLocked` then forms the exact name for a role add (no holder remains) and accepts an explicit name; with a kept holder it numbers a role name or refuses an explicit one as today, so seatengine's `checkName` still refuses while the holder is provably live.

What this can now let through, and its bound: an add kills the present pane of a dead-read holder, a shell-only pane included (D1's Consequences), and a holder misread as dead under D1's residual cases is killed with its engine; it touches only the one strand holding the exact asked-for name in this worktree's state, never while the pair is paused, never a hidden or live one, and never one within 30 seconds of its pane's launch.
Creel addresses receivers by strand name, so mail and injects queued for the dead holder go to the new strand of that name.

`internal/reedengine/doc.go` gains the reclaim rule and its bounds beside the naming rules.

`TestAddStrand_ReclaimsADeadHoldersName` is a table over `AddStrand` on a fake tmux scripted as in `TestAddStrandUnless_NoNamedStrandAdds`, with `e.paneAge` stubbed and the holder an own-window strand named `tc:tslug:fix`:
- a role add for `fix` whose holder's pane is absent gets `tc:tslug:fix`, and the holder leaves the state and `Directory`;
- the same by explicit name;
- a holder whose pane holds only its shell (`bash` over `/bin/bash`) aged 31s is reclaimed and its pane is killed (one `kill-pane` naming it), which is the #799 and #811 shape of a respawn after every pane died;
- a live holder (`claude` over `/bin/bash`) keeps the name: the role add gets `tc:tslug:fix-2` and the explicit add is refused with `strandNameLocked`'s "already held" message;
- a hidden holder keeps the name;
- a shell-only holder aged 29s, inside the grace, keeps the name;
- a dead holder while a live pause entry covers the pair (geometry `PauseRecordPath` and `NameSlug` set, the record written with `pauserecord.Update`) keeps the name;
- a dead holder with a child strand keeps the name;
- a dead holder of the numbered name `tc:tslug:fix-2` is not reclaimed by a role add for `fix`, which gets `tc:tslug:fix`;
- with the session-wide listing (`answerWindowPanes`) scripted to fail, the add returns that error and the holder stays in the state.
`TestStrandNameLocked_ExplicitNameHeldRefuses` is unchanged, since `strandNameLocked` itself does not change.

**ImpactSummary:** One new pre-step on reed's ordinary add path, reached by every respawning caller (shuttle, seatengine, orch, loom), that removes only a dead, visible, childless, unpaused holder of the exact asked-for name; the `--if-absent` and `unlessName` paths are untouched.

**Commit:** 6: a reed add reclaims the name a dead strand holds
