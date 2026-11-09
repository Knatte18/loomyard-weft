```yaml
lens: interactions
outcome: reported
```

Interactions lens: I ran each pair of decisions that share state, then checked each acceptance criterion and constraint against the decisions it touches.

Pairs checked:
- D1 × D2 (lock, pid follow, start-time rule, resume rewriting the start): held.
- D1 × D10 (non-zero exit on `stopped`, `interrupted` or cap stops the shortcut; `owned_here` exits 0): held.
- D2 × D3 (a resume at or before `owner` restarts at `reconcile` and re-runs the later steps over the stored inputs): held.
- D2 × D4 (owner-write-stood list kept across a resume; `--discard` ends the cover, as D4's bound states): held.
- D2 × D5 (a re-run of `pause` skips slugs already listed; a slug the operator resumed stays resumed): held.
- D2 × D7 (`--no-orch` survives a resume from a plain shell; `lym beam start` starts the orch later): held.
- D2 × D10 (`--here` takes the record's request; `--keep-local` and `--drop-local` rules unaffected): held.
- D2 × D12 (`--archives` reads no record, so D2's other-subjects refusal does not reach it): held.
- D3 × D4 (a held-back project ends the record `stopped`, so its subject stays covered): held.
- D3 × D5 (`pause` before `owner`; held-back runs are not paused and stay covered by D4): held.
- D3 × D6 (`pause`, then `owner`, then `deploy`; a failed deploy stops the record with owner and pauses standing): held.
- D3 × D10 (take-up keys on `check` not done; a held-back stop after `check` takes no worktree up): held.
  The first `lym s` in a completed sibling subject still exits non-zero and needs a second `lym s`; D10 and AC 296 intend this.
- D4 × D10 (both resolve the covered beam through `hubgeom.ParentOf` from the subject): held.
- D4 × D11 (D11 runs only when the subject is not under adopt; the covered dobby runs none of its steps): held.
- D5 × D6 (after the owner write, D4's cover ends; adopted runs stay paused, and only a front task can start, as D6 states): held.
- D5 × D8 (the green end lifts no pause, which matches D5's "check never lifts"): held.
- D5 × D11 (the old owner's `OwnerCaller` entries are converted to `AdoptCaller` in place on a later adopt back): held.
- D6 × D10 (`lym s` after a stop at `deploy` resumes at `deploy`, with no `beam start` until done): held.
- D7 × D8 (the dobby and orch come up only in the `beam` step; a failed beam step is the warning's way back): held.
- D7 × D10 (`lym s` passes `--no-orch`, and `beam start` then starts the orch): held.
- D9 × D2 / D3 (status shows `running`, `interrupted` and `stopped`, with the way forward for each): held.
- D11 × D12 (the `archive-owner/` prefix is written by D11 and read by D12; the end envelope names `--archives`): held.
- Acceptance criteria D1–D4 and D6–D12, and the Constraints, against the decisions they touch: held.
- Acceptance criterion D5, line 287, against D5: broken (see F1).

Findings:

F1 — LOW, consistency, `_lym/discussion/decision-record.md:287` (AC D5) against D5 at `decision-record.md:128-133`.
The criterion says "each adopted run, live here or not, and each released task gets an `adopt` pause entry".
D5 exempts two cases the criterion drops: "A slug already holding an entry of another caller keeps it and gets none" (line 133), and released tasks that are front tasks (line 130).
A test written to the criterion as stated would assert entries that D5 forbids.
Fix: add both exceptions to the criterion.

Tool calls: none.
