```yaml
lens: interactions
outcome: reported
```

Lens `interactions`: I checked the pairs of decisions that act on the same state. **2 BLOCKING, 5 MEDIUM, 3 LOW.**

**Pairs checked**
- D1×D3 (Record-Write seeds the Sketch as the raddle run's record): held.
- D2×D9, `tapestry` shape (sketch label dropped, recipe changed): held.
- D2×D9, `project` and `beam` shapes (sketch label on the new project): unstated. LOW, F9.
- D2×D6 (a part's type label could be `sketch`): broken. MEDIUM, F5.
- D3×D9, darn parts (part record seeded at Record-Write): held.
- D3×D9, project and beam (pawl's Record-Write seeds the whole Sketch as the beam record): unstated. MEDIUM, F6.
- D5×D9 (parts split by decision id; `Added after Discussion` entries have no id): broken. MEDIUM, F4.
- D6×D8 (slug and dependency checks run at gate, approval and filing): mostly held. LOW, F8.
- D7×D8 (return from Raddle-File to Raddle-Write re-commits the cold proposal): broken. MEDIUM, F3.
- D7×D14 (cold against approved): broken through F3.
- D8×D9 (re-check inside the filing write): held.
- D9 order × D11 (beam created before the proposal is stored): broken. BLOCKING, F2.
- D9 idempotence × D9 later steps (beam create, proposal store, notice): broken. MEDIUM, F2.
- D9 `beam` × D10 (beam create while the raddle's own run is live): broken. BLOCKING, F1.
- D9 status clear × the run's status seam on a Stuck after the board write: broken. MEDIUM, F7.
- D9 `tapestry` shape × D10 (next start clears the done raddle run): held.
- D10 × batten Land: held. `childFinalized` reads a run with no Finalize as landing no code, and land writes no draft.
- D12×D13 (diff size comes from Finalize; motif and raddle runs have no size): held. Their own cost is outside the fit, which is acceptable.
- D12×D14 (`tapestry` shape reuses the Sketch slug, which also carries the motif and raddle runs): needs a split by recipe. LOW, F10.
- D7×D9 (auto-approve files released tasks): held, intended.
- Every AC and constraint against its decisions: held, except AC D9 "repeated Raddle-File writes nothing" (F2) and AC D11 (F2).

**Findings**

F1 — BLOCKING — D9 × D10: the `beam` shape runs `lym beam create` from inside the live raddle run, so the clear D10 relies on cannot run.
- D10 makes `lym beam create` clear only a *done* record-only run.
- While Raddle-File runs, the raddle run holds the slug's run lock, so `clearMotifRun`'s `TryAcquireWriteLock` gets `free=false` and clears nothing.
- `lym beam create` then meets a slug with a live run and pair.
- Fix: run beam create after the raddle run is done, for example from batten after its Land, or have the clear treat the calling run as its own. State that order.

F2 — BLOCKING — D9 order × D11 and D9 idempotence: the beam's orch can start before the proposal is stored, and a retry skips the steps after the board write.
- D9's order is: board write, then `lym beam create`, which starts the pawl first pass whose BeamUp brings up the orch, then store `raddle-<slug>.yaml`.
- D11 mails the orch only at BeamUp's first start, and only when a stored proposal exists. A fast BeamUp finds none, and the hand-off is lost for good.
- D9's retry rule ("finds the entry no longer held and writes nothing twice") also skips beam create, the proposal store and the notice when a crash fell between them.
- Fix: store both proposals before or in the same step as the board write. Make each later step idempotent on its own: beam exists, proposal stored, notice posted.

F3 — MEDIUM — D7 × D8 × D14: a return from Raddle-File to Raddle-Write re-runs the gate-pass step, which commits the cold proposal again.
- The recommit overwrites the operator-corrected file as the "cold" one, so D14's cold-against-approved comparison loses the real cold proposal.
- D8's "created after the cold proposal" baseline moves with it.
- D7 also ends the session after approval, and does not say how a return re-opens it.
- Fix: commit the cold proposal once per run, at the first gate pass only. State that a return re-opens the session in the run's mode.

F4 — MEDIUM — D5/D9 × the discussion spec: `### Added after Discussion` entries in the Sketch carry no id, so the D5 parts cannot carry them, and D9's splice drops them.
- These entries are parent decisions, and they win over the entry body.
- Fix: copy every such entry into every part record, or have the proposal assign them by title.

F5 — MEDIUM — D2 × D6: D6 accepts any type label in the board's vocabulary, and that vocabulary always holds `sketch`.
- A part typed `sketch` with recipe tapestry or darn passes the gate and approval.
- The board then refuses it at filing under D2's rule.
- Fix: D6 refuses `sketch` as a part's type label.

F6 — MEDIUM — D3 × D9, `project` and `beam` shapes: the entry becomes a project that keeps its `discussed` flag and the whole Sketch.
- Under D3, pawl's Record-Write then seeds the whole Sketch as the beam's record, which becomes its project record.
- The record does not say whether that is intended.
- Fix: say so, or clear the flag on the project.

F7 — MEDIUM — D9 × the run's status seam: on any non-Done transition, `newCommitStatusSeam` writes `<state> · <producer>` to the entry.
- If Raddle-File goes Stuck or Halt after its board write, for example on a failed beam create, the seam writes a run status back onto the new task, project or done entry.
- That re-locks the entry or overwrites `done`.
- Fix: D9 or D10 states that the status write stops once the filing write has landed.

F8 — LOW — D6 × D8: D6 allows a `depends_on` on a done entry, but D8 requires every outside dependency to still be on the board.
- A done dependency pruned between gate and filing fails the re-check, though it is satisfied.
- Fix: D8 counts a dependency as satisfied when it is done or pruned as landed.

F9 — LOW — D2 × D9, `project` and `beam` shapes: the record does not say whether the `sketch` label is dropped when the entry becomes a project.
- D2's rule covers tasks only.

F10 — LOW — D12 × D14: in the `tapestry` shape the filed part is the Sketch slug itself, so the slug's history also holds the motif and raddle runs.
- D14 must count only the tapestry run's cost.

**Tool calls**
1. `internal/battencli/land.go` (lines 130–170) and grep of `Finish` in `internal/loomcli/wiring.go`: to learn how Land and the finish hook treat a run with no Finalize. My context had only the subagent's summary.
2. `internal/loomcli/wiring.go` (lines 250–290) and grep of `childFinalized`: to learn the order of the status seam's writes against Finish, which F7 rests on. My context lacked it.
