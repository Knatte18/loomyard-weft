```yaml
lens: interactions
outcome: reported
```

Interactions lens: I checked every pair of decisions in the record that touch the same state, and every acceptance criterion and constraint against the decisions it touches. Two pairs break outright (BLOCKING), and five more are MEDIUM.

**Pairs checked**
- D2 × D4 (push rejection): held.
- D2 × D7 (a push owed under a hold): held.
- D2 × D1 (the act rule's "push owed" input): broken, see finding 3.
- D3 × D1 (pause intent against `halted`): held. Which state wins is unspecified (LOW).
- D3 × D11 (cancel intent keep or remove): held.
- D4 × D5 (machine trailer against renames): broken, see finding 4.
- D4 × D8 (takeover commit on a shared batten records branch): broken, see finding 1.
- D4 × D6 × D8 (the order of takeover and catch-up on the adopter): broken, see finding 2.
- D6 × D11 (pair removal against orphan teardown): held.
- D7 × D8 (one `hold` key against several subjects and a `pause` hold): broken, see finding 5.
- D7 × hibernate's local pause × D1 (inputs only from tracked records): broken, see finding 3.
- D7 × D3 (legacy adopt entries become intents, while D8 lets runs go on unasked): held, as a stated exception.
- D8 × D9 × D10 (orch start, resume, restore): held.
- D9 × D10 (the mirror at handover against D9 ending every process): broken, see finding 6.
- D10 × D6 (restore by length after the two copies forked): LOW, see finding 9.
- D11 × D2 (the claim "no push owed at resume"): LOW, see finding 8.
- D4 × D8 against the `PATTERN-owner-machine` Finalize constraint: broken, see finding 7.
- D5 × D8 (registry write at adopt): held.

**Findings**

1. **BLOCKING — D4 × D8.** The takeover commit lands on a records branch that many runs share, so a beam adopt can stop runs that never moved.
   - Batten records live on the parent's records branch, for example `main-weft` for root's runs. A beam's own pawl record sits there as well.
   - When a beam is adopted, D8 step 2 writes a takeover commit to that shared branch, which the parent's owner still writes to.
   - The old machine's next push of any root run is then rejected over a commit naming another machine. D4 stops "the run", ends that root run's processes and resets the branch, even though that run never moved.
   - D4 assumes one run per branch with one writer. That fails whenever the parent and the beam have different owners.
   - Evidence: state map 2.2 (the prime's runs are recorded in the prime weft's `_lym/shed/<slug>/`); D4's rejection rule; D8 step 2.

2. **BLOCKING — D8 steps 2 and 3, with D6 and D4.** The takeover commit comes before the catch-up, so the adopter can hand over its own adopted run.
   - When the adopter last owned a run, it may have left local commits that the other machine later superseded.
   - Step 2 then makes the takeover commit on top of that stale local branch, and its push is rejected over the other machine's commits.
   - D4 then ends the adopter's processes, resets the branch, and "stops acting on the run" that the adopter just took over.
   - D6 step 3 likewise assumes local commits sit on top of origin's tip and never covers local history that has diverged from origin on an adopted run.
   - The fix is to reset each moved run to origin first, archiving the local tips, and only then write the takeover commit.

3. **MEDIUM — D1 against D2 and D7.** D1 says the act rule reads only tracked inputs, yet it answers `wait` on "push owed and origin unreachable", which is local state.
   - D7 keeps hibernate's pause entries in the local pause record, but D1's act rule never reads them. So either a hibernate pause holds nothing, or the act rule has a local input.
   - The D3 criterion "a run whose `.lym` is removed gets the same act answer" also fails for a run with a push owed.

4. **MEDIUM — D4 × D5.** D4 does not resolve the trailer's name through the registry.
   - After `lym machine name`, a rejection over this machine's own earlier commits, which carry its former name, reads as another machine.
   - D4 then archives and resets instead of replaying.
   - D4 should compare the trailer through D5's registry.

5. **MEDIUM — D7 × D8, and D7 against tender.** Bolt has one top-level `hold`, but there can be several holds at once.
   - Two adopts of different beams, possibly from two machines, would overwrite each other's hold.
   - An adopt over a standing operator `pause` hold replaces it, and a green check then clears it, silently lifting the operator's pause.
   - The `PATTERN-dobby-runs` change text also omits `lym tender resume` as a way to clear a `pause` hold, which D7 grants.

6. **MEDIUM — D9 × D10 and the `PATTERN-child-processes` constraint.** D9 can kill the very mirror it starts.
   - The mirror runs at handover as a detached child registered in runproc.
   - D9 ends every process registered for the subject, which would include that mirror.
   - The order (mirror first, then end the processes, excluding the mirror) is not stated.

7. **MEDIUM — D4 and D8 against the `PATTERN-owner-machine` constraint** that the rest of a Finalize past its parent-side merge finishes on either machine.
   - D4 stops the old owner when its push is rejected, even mid-Finalize after the merge.
   - D8 has the new owner resume from the last pushed transition, which may come before the merge that has already happened.
   - The record says the constraint still holds but never says how D4 and the act rule treat a run in `landing`.

8. **LOW — D11.** "With D2 no push is owed at a resume" contradicts D2, under which a push stays owed while origin is unreachable. The tender behaviour is fine; only the stated reason is wrong.

9. **LOW — D10 × D6.** Restoring when origin's mirror "holds more" judges by length, which misreads two copies of one transcript that both grew after they diverged, as the stale holder in A7 might. Say which copy wins: origin's, after D9 ends the holder.

No further tool calls were made.
