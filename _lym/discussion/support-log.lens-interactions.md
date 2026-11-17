```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I checked the pairs of the record's decisions that act on the same state, plus the acceptance criteria and constraints against the decisions they touch.

**Findings**

1. **MEDIUM — D6 × D2/D3: the clean-tree check runs only at the start.**
   - D6 refuses a dirty tree at the start, then guards only HEAD before each write.
   - D2 measures packages one after another and builds one package ahead, so a run lasts minutes.
   - An edit made during the run, without a commit, goes into later packages' coverage builds. HEAD stays still, so those maps still get written, stamped with the HEAD commit and its hashes.
   - So D6's rationale ("a clean tree makes the commit stamp true") is outrun by D2's long run.
   - Fix: re-check that the tree is clean, or the package's file hashes, before each build or write, alongside the HEAD check.

2. **MEDIUM — D4 × D5 × D7: a guard kill beside other tests has no clear rerun rule.**
   - D4 records a guard-killed test as failed and goes on.
   - D5 reruns alone only a test that "fails or times out beside others".
   - D4 also keeps the guard's pressure trigger, which kills the largest tree. While tests run side by side, that may not be the test that caused the pressure.
   - So a test killed only because of its siblings gets a failed entry, and D7 makes the call exit non-zero, with no alone-rerun.
   - D7's "timed out" is equally unclear: a test that times out beside others and passes alone is non-zero by D7's wording but passing by D5's.
   - Fix: say whether a guard kill beside others counts as a D5 failure, and limit D7's non-zero exit to outcomes after the alone-rerun.

3. **LOW — D2 × D6: the one-ahead build is not handled when HEAD moves.**
   - When D6 ends the call on a HEAD move, the next package's build that D2 started is never handled: neither cancelled nor waited on.
   - That conflicts with `PATTERN-child-processes`, which says a starter waits on what it spawns.
   - Fix: state that the pending build is cancelled and waited on.

4. **LOW — D7 × D2 × the `PATTERN-test-speed` constraint: failed-build behaviour changes but the record lists only a verb rename.**
   - D7 says a failed build "keeps no old entries".
   - Today's rule, in the `PATTERN-test-speed` background, is "A failed build keeps the map's entries and is not retried at the same HEAD".
   - The Constraints bullet says the background changes only its wording from the verb to the flag. The background's planning sentences (selector, self-check, audit round) also go under D2.
   - Fix: the constraint should say the coverage-map section is rewritten for D2 and D7's behaviour, not only renamed.

5. **LOW — D7: "run log" is not the same as a run record.**
   - D7 says a map run writes a run log.
   - The gate's run records (`gateslot.ReadRunRecords`) feed `lym gate report` totals and timing.
   - The record does not say whether a map run writes one. If it does, minutes-long coverage runs skew the per-key timing totals.
   - Fix: say whether a map run writes a run record, and if it does, under which key.

6. **LOW — Acceptance (D1): only one of the two tiers is tested for the `go-json` refusal.**
   - D1 refuses when tier 0 *or* tier 1 is not `go-json`, but the criterion covers tier 0 only.
   - Add the tier 1 case, or word the criterion for either tier.

**Pairs checked**

| Pair | Result |
|---|---|
| D1 × D6 (default diff scope vs clean tree) | held |
| D1 × D7 (refused `--gate` and carry vs no green record) | held |
| D1 × D9 (target form in the redundant refusal) | held |
| D2 × D6 (one-ahead build vs HEAD-move stop) | broken (LOW, #3) |
| D2 × D7 (whole replace vs failed build keeping no entries) | held internally; PATTERN background mismatch (#4) |
| D2 × D8 (whole replace vs last write wins) | held |
| D3 × D4 (per-tier env and parallel) | held |
| D3 × D7 (prebuilt `lym` for tier 1 vs slot hold) | held |
| D4 × D5 (guard kill vs alone-rerun) | broken (#2) |
| D4 × D7 (side-by-side clamp vs max-of-tiers slots) | held |
| D5 × D7 (alone outcome stands vs exit rule on timeout) | broken (#2) |
| D6 × D2/D3 (start-only clean check vs long run) | broken (#1) |
| D6 × D8 (commit stamp vs location) | held |
| D7 × D8 (failed build map vs last write wins) | held |
| D8 × the redundancy step (readers under `internal/state` lock, no build lock) | held |

**Acceptance criteria and constraints**

| Item | Result |
|---|---|
| D1 refusal criteria | partial (#6) |
| D2, D3, D5, D8, D9 criteria | held |
| D4 criteria | held, but the guard-kill criterion conflicts with the D5 rerun (#2) |
| D6 criteria | held, but no criterion covers a mid-run edit (#1) |
| D7 criteria | held |
| `PATTERN-gate-slot` (slots, `-trimpath`, green only on its own record) | held |
| `PATTERN-test-speed` | inconsistent (#4) |
| `PATTERN-child-processes` | at risk (#3) |
| `PATTERN-refusal-way-forward` | held |
| `PATTERN-tolerant-records` | held |

Tool calls: none; my context held everything this lens needed.
