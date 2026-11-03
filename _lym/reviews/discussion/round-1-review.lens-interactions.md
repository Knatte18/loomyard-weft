```yaml
lens: interactions
outcome: reported
```

Task: interactions lens. Check every pair of decisions that act on the same state, plus the acceptance criteria and constraints against the decisions they touch.

**Pairs checked**
- D1 × D3 (order key; status strand at key one past runs): held. Status at 1 opens after runs at 0, whichever is added first.
- D1 × D4 (cap removed; runs strand added beside status strand): held. Runs at 0 opens before an existing key-1 status window. The D4 add would hit no refusal: `addStrandLocked`'s overview check (`internal/reedengine/strand.go:404-410`) is what D1 removes.
- D1 × D2 (ascending keys; fallback to the lowest-index overview): held with distinct keys. The lowest index is key 0, the runs strand. Broken during the D5 tie window, see F1.
- D1 (`ReplaceStrand` refusal removed) × D5 (next loom start replaces an own-window status strand with an overview one): held. Before D1, `ReplaceStrand` → `addStrandLocked` refused while the runs strand held the overview (`internal/reedengine/replace.go:81`, `strand.go:404-410`). D5 heals only because D1 removes that refusal.
- D1 (`UpdateStrand` refuses an order-key change) × D5 (missing key reads as zero): held. No path updates a display's key, and the old-key strand is healed by replace, not update.
- D1 tie rule × D5 bound (old-key status ties runs at 0): held as an ordering statement. D5 names add, replace and relaunch.
- D2 × D5 (an old own-window status strand counts as an agent window): held. D5's bound states it, and `relevantWindow` treats an own-window pane as an agent (`internal/reedengine/ownwindow.go:442-446`).
- D3 × D4 (status no longer reads strands; runs no longer replaces status): held. Neither strand's ensure removes or reads the other.
- D3 × D5 (recipe-titled overview spec used by the healing replace): held. `ensureStatusStrand` builds the D3 spec for both replace and add (`internal/loomcli/sharedbootstrap.go:300-316`).
- D3 (KeepViewer branch removed) × Constraint "never selected when it opens": held. `openOwnWindowLocked` never selects an overview window (`internal/reedengine/spawn.go:280-286`), which covers what KeepViewer guarded.
- D3 × the `FollowRun` call after `ensureStatusStrand` (`internal/loomcli/start.go:559-563`) × D2: held. With no live agent, the follow lands on the key-0 beam overview, as D2's rationale intends.
- D1 × Constraint `PATTERN-named-constants` × D3 (status key derived from runs key): held.
- D4 acceptance criterion ("leaves the status strand, its pane and its window untouched") × D1 (runs window opens before it): held. The window's identity is untouched, and only its index shifts.
- D5 acceptance criterion × D1/D3: held for the own-window case. The no-status-strand case is covered by D3's add-spec criterion.
- D2 acceptance criterion × D5 tie state: not covered, see F1.

**Findings**

F1. Severity LOW, class consistency.
- Location: `decision-record.md`, D5 Bound, against D2's Rationale.
- Summary: D5's tie state breaks D2's premise that "the lowest-index overview is the beam overview", and the record states only the window order, not the focus result.
- Case: an older lym's `lym beam create` pass ran the pawl `lym loom start`, which left the status strand in the overview with no key. After the upgrade, `lym beam start` runs D4 and adds the runs strand at key 0 beside it. The two tie at 0, and the runs window opened last, so it sits after the status window.
- Result: D2's fallback (`FollowRun`, attach, a viewed window ending) then lands on the pawl checklist, not the beam overview, until the next loom start replaces the status strand.
- Fix: D5's Bound names this focus consequence beside the order consequence.
- Evidence: decision-record.md D2 Rationale and D5 Bound; `internal/orchcli/runs.go:88-91`, the replace path D4 removes; `internal/reedengine/ownwindow.go:456-471`.

No tool calls made.
