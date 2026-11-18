```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the round-1 change set agrees with its surroundings (fix-consistency lens).

Changed text checked: D1 config_dir, D2 entry/prune, D3 steps 2–4 and reset, D4 resolution and told geometry, D5 settings and Bounds, D6 fallback and Facts, D7 rewrite, D8 known directories, D10 wiring, Constraints, Auto-mode assumptions, Open risks, Acceptance criteria, Notes, and the support-log ledger and alternatives.

1. **MEDIUM. Acceptance criteria, Review bullet: "Review: the task's landing notes state how the second account is wired and the manual check's steps."**
   - The rewrite dropped "D10" from the bullet, so no criterion names D10 any more.
   - The review's directive-6 verdict found every id from D1 to D10 named as a whole word by at least one bullet, and this edit breaks that.
   - Evidence: the old bullet "Review: D10's manual check … with its outcome". No other bullet names D10.

2. **MEDIUM. Constraints, `PATTERN-shuttle-provider-seam` line: "removes the strand only through the stop verb's relaunch mode, as the pattern's Shuttle stop section requires."**
   - D7 step 1 has the relaunch mode remove the strand while it "records no outcome and leaves the run record live".
   - The pattern section says the stop verb "settles the run's record first". It names the failure: a removal that "leaves the record saying the run is live, and an attach probe or a Wait then reads a strand that is gone".
   - Between steps 1 and 4 the record says live and the strand is gone, which is that case. So "as … requires" does not hold.
   - The review offered a declared `## PATTERN changes` bullet as the alternative. The record neither has one nor says how the record is settled for an attach probe during the gap.
   - Evidence: `pattern/PATTERN-shuttle-provider-seam.md:78-82`.

3. **LOW. D6, first paragraph: "the resume check, which shuttle runs before `Prepare`".**
   - D7's relaunch now runs `Prepare` (step 2) before the resume check (step 3). D6 states the before-`Prepare` order without exception.
   - D6 also describes its fallback as "a shuttle run through D7's in-run relaunch with a fresh launch". D7's steps describe only a resume launch, so the fresh variant has no steps.

4. **LOW. Goal: "pinnable per hub or for the prime and beams, and switched to another account when its own runs out."**
   - After the fix a hub-pinned worktree waits a limit out (D7, Auto-mode assumption 1). The Goal still says every worktree switches.

5. **LOW. support-log.md Interview, Turn 4 entries.** These still say:
   - "D7 relaunches inside the same run, and the stop-verb constraint is gone";
   - "a pinned worktree switches while its pin is out";
   - "board-scope F2 … kept as an Open risk for the run's parent".

   Round 1 reversed all three: the relaunch goes through the stop verb, only the beams pin switches, and Q11 is an auto-pick. The new Round 1 entry under Review rounds records the change, but nothing next to these lines marks them as superseded.

6. **LOW. D8: "`~/.claude`, compared as D1 says".**
   - D1 now compares directories with links resolved and refuses only a listed directory that does not resolve.
   - D8's known directories include the `env` module's value and `~/.claude`. Neither is a listed directory, so D1 says nothing about one that does not resolve.

No further tool calls made.
