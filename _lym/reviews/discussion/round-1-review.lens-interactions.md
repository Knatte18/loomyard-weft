```yaml
lens: interactions
outcome: reported
```

Task: interactions lens. I checked pairs of decisions that act on the same state, using only the inherited context, with no tool calls.

**Pairs checked**
- D2 × D9 (`sketch` label on a raddle task, then the entry marked done): held.
- D2 × D10 (raddle entry keeps the label; its finish skips the end-as-Sketch finish): held.
- D2 × D9 bound (the raddle agent's own promote or upsert stays refused by `checkTaskCreation`): held.
- D3 × D9 (Record-Write seeds filed darn parts from `decision-<slug>.md`, and the pawl run from the project record): held.
- D3 × D2 (the raddle entry stays discussed, so Record-Write seeds the Sketch): held.
- D5 × D8 (new slugs differ from the Sketch's; the slug-free re-check): held.
- D5/D6 × D9 (coverage checks against what part records carry): broken for PATTERN changes (F2).
- D6 × D9 tapestry shape (one part builds every decision; its record is the Sketch whole): held.
- D6 package-overlap × D8 dropped `depends_on` (only dependencies on outside entries are dropped): held.
- D7 × D8 (hash binding after a filing failure returns the run): held. The waiting notice after that return is broken (F3).
- D7 cold proposal × D9 step 1 × D14, key on: broken, the cold proposal is undefined (F4).
- D7 key × D9 (key read per row; Raddle-File runs no D6 check): broken (F6).
- D8 filing failure × D9 step idempotence (store and records written before the board write): broken (F1).
- D9 step 3 (entry done mid-run) × the run's later halt or status writes: broken (F5).
- D9 × D10 × D14 (Land prunes the Sketch; the store keeps the copy): held, apart from F1.
- D9 × D11 (store written before the beam step; hand-off keyed on the filed project): held, apart from F1.
- D12 × D8 return (new session id gives a new usage file): held.
- D12 × D6 re-prompt (same live session, one end): held.
- D12 × D13 (Seed's recipe and profile, Finalize's size): broken for runs with no Finalize (F7).
- D6 attempts × D8 returns: held, and matches the D6/D8 criteria.
- Constraint `PATTERN-loom-contract` × D7 bound (verbs refuse by row): held.
- Acceptance criteria D7, D8 and D9 against D7–D9: consistent, except that the D9 retry criterion does not cover F1.

**Findings**

F1. BLOCKING, design. Location: D9 steps 1–3 × D8's filing-time failure.
- Summary: Raddle-File writes the raddle store (step 1) and the part records (step 2) before the board write that runs D8's re-check (step 3). When that re-check fails, the row returns to Raddle-Write.
- The revised and re-approved proposal then reaches a Raddle-File whose steps are "skipped when [their] result is already there". So step 1 keeps the stale approved proposal, and step 2 leaves `decision-<old-slug>.md` records for slugs that were never filed.
- What breaks:
  - D10's `lym board raddle <filed slug>` cannot find the slugs actually filed.
  - D11 sends no hand-off when the revision renamed the beam project.
  - D14 compares the wrong approved proposal.
  - Orphaned stored records sit under slugs a later entry could take.
- Evidence: `_lym/discussion/decision-record.md` lines 101–102 (failure at filing returns to Raddle-Write), 108–114 (step order and skip rule), 133 (D11 trigger), 153 (D14).

F2. MEDIUM, design. Location: D6 coverage checks × D5/D9 PATTERN changes.
- Summary: D6 requires every decision, Acceptance criterion and Open risk to be carried by at least one part, but sets no such check on PATTERN changes. D5 and D9 still give each part only the PATTERN changes it carries.
- A Sketch PATTERN change that no part carries is dropped silently at filing. One carried by two parts is landed twice.
- Evidence: `decision-record.md` lines 58, 68–69, 112.

F3. MEDIUM, design. Location: D7 waiting notice × D8 return.
- Summary: `raddle-propose` posts "a proposal waits for the operator" only on its first passing call in a run, and the criterion pins "on its first pass only".
- D8's filing-time failure returns to a new interactive Raddle-Write session whose revised proposal "needs a new approval". No notice tells the parent orch or operator that it waits, so the run sits in an unannounced pane.
- Evidence: `decision-record.md` lines 84, 91, 102, 196.

F4. MEDIUM, design. Location: D7 × D9 step 1 × D14, key on.
- Summary: the cold proposal is defined only by `raddle-propose`'s first pass, which runs only with the key off.
- In auto mode, D9 step 1 still stores "the cold proposal" and D14 reports whether the approved proposal differs from it. After a D8 return in auto mode, the first and the last gate-passing proposals also differ.
- No decision says what the cold proposal is in auto mode.
- Evidence: `decision-record.md` lines 82, 84, 110, 153.

F5. MEDIUM, design. Location: D9 step 3 × the rest of the run (steps 4–5, a halt, the finish).
- Summary: the run's own entry is marked done in step 3 while the run goes on. `SetRunStatus` skips a done entry.
- So a halt in step 4 (`lym beam create` fails) or step 5, or after 2 returns, can never show on the board. The board reads done while the run waits halted for its parent.
- D9's Bound claims the skip as a benefit and states nothing about what it hides.
- Evidence: `decision-record.md` lines 118–123; `internal/boardengine/board.go:203`.

F6. LOW, design. Location: D7 key × D9.
- Summary: the mode is a setting looked up by key (Constraints, `PATTERN-shed-recipe-registry`), so it can differ between Raddle-Write and Raddle-File.
- A run approved or proposed with the key off, then filed with the key on, skips the hash match. Raddle-File re-runs only D8, never D6, so a `proposal.yaml` edited after the session ends is filed unchecked.
- D7's Bound does not state this.
- Evidence: `decision-record.md` lines 80, 90, 101, 114, 162.

F7. LOW, design. Location: D12 × D13.
- Summary: D13 fits a fixed cost plus a cost per 100 changed lines for every recipe and profile. D12 writes the landed size only in Finalize, and record-only recipes such as raddle and motif have no Finalize row (D1).
- Neither decision says how a run with no size record enters the fit, or whether it is left out.
- Evidence: `decision-record.md` lines 24, 143, 147.

**Tool calls**: none; the inherited context held every file the lens needed.
