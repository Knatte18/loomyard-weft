```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I checked every pair of decisions in the drift-rework record that act on the same state, and the criteria and constraints against them.

**Findings**

1. **BLOCKING — D4 × D9/D10/D11 (Drift-Recheck's skip rule), and the D4 criterion:** Drift-Recheck's "Done at once when the warp head equals the last compared head" lets a Stuck Drift-Recheck pass on resume without checking or judging again.
   - Evidence: Drift-Recheck finds an unexpected cell, or gets a `does not match` ruling, and is Stuck. The parent resumes without a code change, or amends under D11 and resumes.
   - Nothing says whether a Stuck compare records its head as "last compared". If it does, the head still equals it on re-entry, so the row is Done. The regression is never re-checked, and an amendment's dropped rulings are never judged again.
   - Fix: allow the skip only when the last compare at that head passed with every cell ruled `matches`. Record the passing head, not merely the compared head.

2. **MEDIUM — D6 × D7, and the D7 criterion:** "A run with no `expected.yaml` past Drift-Expect is checked against an empty expectation" contradicts D6, which says "every containment check checks the seal before it diffs".
   - Evidence: a run in flight that is past Drift-Expect has no seal. D6's checks fail before any diff, so the run is Stuck even with zero drift.
   - The D7 criterion "stops on any drift" and Open risk 5's "stops on any drift" both assume the diff runs.
   - Fix: when no seal record exists and Drift-Expect never ran (or was Done at once), skip the seal check and use the empty expectation. Keep the seal failure for the case where a seal exists but does not match.

3. **MEDIUM — D6 × D11:** D6 says a failed seal check is cleared only by "a parent amendment (D11) or a restored file", but an amendment cannot clear it.
   - Evidence: D11 says an amendment "never edits `expected.yaml` or the seal". The seal check compares the file's hash with the sealed hash and with the committed copy, so it still fails after any amendment.
   - Fix: drop the amendment as a way out of a seal failure (the restored file stays the only one), or give D11 a re-seal form with its own bound.

4. **MEDIUM — D11 × D4/D5/D8:** amending at a Stuck Drift-Expect is undefined.
   - Evidence: Drift-Expect is Stuck when the `expectation` entry is exhausted, which means `expected.yaml` is invalid and unsealed. An amendment adds coverage but neither repairs nor seals the file, and D8's `check` refuses before a seal exists.
   - D11 also leaves open whether amendment entries are checked by D5's parser rules, such as the case existing in the baseline.
   - Fix: drop Drift-Expect from where `amend` is allowed, or define what an amendment does before the seal. State that amendment entries go through D5's validation.

5. **MEDIUM — D2 × D6:** in `command` mode nothing checks the seal.
   - Evidence: D6 checks the seal only in "every containment check (D7, D8)". D7 is lines mode only, and D8 passes in `command` mode. Yet the D2 judge rules against the expectation, so an edited `expected.yaml` goes unchecked there.
   - Fix: run the seal check in Drift-Verify whatever the mode, before the compare or judge.

6. **MEDIUM — D2 × D9 (the `judgement` entry and its parser):** the judge's output is defined only for lines mode, yet D2 reuses the judge.
   - Evidence: D9's `judgement.yaml` has one ruling per cell id, and its gate entry refuses a missing or unknown cell id. In `command` mode there are no cell ids, and the rulings are per case with a third value, `unexpected`.
   - Fix: give `command` mode a per-case form of `judgement.yaml` and its gate check, or state the format and validation for both modes in D9.

7. **MEDIUM — D7 × D8 (shared scratch capture directory):** concurrent checks can overwrite each other's capture.
   - Evidence: D7 captures under the capture lock. D8's entry and `lym drift check` write "the scratch capture directory under `.lym/drift/`", but D8 never says they take the lock.
   - A Webster fork running `lym drift check` while Merriam's run-level `drift` entry runs would write into, and diff, the same directory.
   - Fix: say that D8's entry and verb take the capture lock (or a lock of their own), or give each check its own scratch directory.

8. **LOW — D6 × D4 (Drift-Expect restart):** the seal needs a commit in the middle of the row, before the producer's commit on exit, and its restart path is unstated.
   - Evidence: if the process crashes after `expected.yaml` is committed but before the seal is recorded, re-entering Drift-Expect could spawn the agent again and overwrite a committed, valid file.
   - Fix: on re-entry, seal an existing committed file that validates and spawn no agent.

9. **LOW — D1 × D10 (stability of occurrence numbers):** occurrence numbers are a weak identity for stored rulings.
   - Evidence: inserting one earlier line with a repeated key shifts the occurrence number of every later line with that key. Their tuples change, so they are judged again, and their cells move to different ids even when the values are unchanged. D10 only re-judges, so this costs judge work but does not break the design.
   - Worth a line in D1's rationale.

**Pairs checked**
- D6 seal × D11 amendment storage (`scope.json`, not the file): held. D6's clearing claim: broken (#3).
- D6 × D4, `Warp-SHA` equal to the baseline head at Drift-Expect (plan writers commit only on the weft): held.
- D6 × D7, no seal on a run past Drift-Expect: broken (#2).
- D6 × D9, the judge cannot touch `expected.yaml`: held.
- D6 × D8, the writer edits `expected.yaml` and the gate catches it: held.
- D6 × D2, command mode: broken (#5).
- D6 × D4, restart of Drift-Expect: unstated (#8).
- D7 × D12, deferred capture before a missing expectation: held if the deferred check comes first.
- D7 × D8, shared scratch directory: broken (#7).
- D7/D8 × D11, amendments in containment: held.
- D9 × D10, judging spawned only for tuples with no stored ruling: held.
- D10 × D11, rulings dropped and judged again: held. Re-entry after an amendment: broken via #1.
- D10 × D1, occurrence ids: LOW (#9).
- D5 × D1, line entries cover only unkeyed cells and `added`/`removed` cover cases: held.
- D11 × D5/D8 at Drift-Expect: broken (#4).
- D2 × D9 output format: broken (#6).
- D2 × D4, Recheck in command mode: held.
- D3 × D4/D8/D13, scope off: held.
- D3 × D15, `partial` read as `full` against the listed values: held.
- D4 Recheck skip rule × resume after a Stuck: broken (#1).
- D12 × D4/D8/D13: held.
- D14 × D2, the `compare` verb removed while command mode stays: held.
- D14 × D3, the `scope` verb removed: held.
- D15 `{skip_file}` refusal × D12 defer path: held.
- `scope.json` writers (seal, amendments, compare head): held, because `amend` is allowed only at a Stuck row.

**Criteria and constraints**
- D4's criterion writes in the broken skip rule (#1).
- D7's criterion clashes with D6 (#2).
- D11's criterion "the seal check still passes after an amendment": held.
- `PATTERN-tolerant-records` (judgement strict, `judged.json` tolerant): held.
- `PATTERN-shed-stop-chain` × D4, removing `on_stuck`: held.

Tool calls: none.
