```yaml
lens: code-tests
outcome: reported
```

**Lens code-tests — per behaviour, the test that pins it, or none**

1. **Fix-review loop cap (`finalize_fix_review_rounds`).** Pinned.
   - The test is "a fix over the caps goes to the fix review… until the rounds run out" in `finalize_gate_test.go`, with rounds = 2.
   - It asserts the session order fixer, review, fixer, review.
   - It also asserts the halt reason ("in 2 rounds", the circling accept hint) and `rec.Halt`, `ReviewRounds` and `Findings`.
2. **Fixer on a fresh turn cap per round.** Pinned.
   - The same test asserts `second.Gate[0].Attempts == FinalizeFixAttempts`.
3. **Review report passed to the fixer (`FixReviewPath`, `fix_review_path`).** Pinned.
   - The same test checks `fix_review_path` empty on the first fixer turn and set on the second.
   - `fixer_test.go` pins the optional-field list.
4. **A rejection the fixer answers, then approval, lands.** Pinned by "a rejection the fixer answers is judged again…".
5. **A review with no verdict runs again with no fixer between, and the last round halts.** Pinned.
   - The "fix review's end decides where an over-cap change goes" table has `wantReviews: 2` rows for timeout, no report and unknown verdict.
6. **Review changed the task branch.** Pinned.
   - The dirty-tree and head-moved rows check the halt reason and a single review session.
7. **Spent fixer still halts as before.** Pinned.
   - The test is "a spent cap halts naming the commits… resume halts again".
   - The seeded-resume row "a failing gate halts as a spent cap" also covers it.
8. **Resume after the review-capped halt.** Pinned.
   - An accepted record lands with no review.
   - A non-accepted record clears the rounds and reviews afresh.
9. **`AcceptFixReview`.** Mostly pinned by `TestAcceptFixReview`.
   - Covered: append plus `Accepted`, a second call writing nothing, refusal on a spent halt, refusal for another entry.
   - Not covered: see F1.
10. **`circling accept` at a Finalize halt.** Pinned.
    - The tests are `TestCirclingVerb_AcceptsFinalizeFixReviewHalt` and the refusal rows for `continue`, `ErrNoFixReviewHalt` and a commit failure.
11. **`HandsBack` drops the reroute-cap clause at Finalize.** Pinned by the changed `RerouteCapAtFinalize` row, now false.
12. **Config key.** Pinned by the "below 1" row in `TestLoadConfig_InvalidKeys`.
13. **`on_reroute` removed from darn and sampler.** Pinned.
    - `darnrecipe_test.go:190` and `samplerrecipe_test.go:74` assert the empty `OnReroute`.
    - Tapestry has no assertion; my grep found no Finalize `OnReroute` check in any tapestry test. The diff changed no tapestry test either. See F4.
14. **Never reroutes.** Pinned by `requireHaltNoLanding` asserting `!ptr.Reroute`.
15. **`FixerAttempts` round-trip.** Pinned by the extended `TestFixerAttempts`.
16. **`runFixer` clears `FixReview` after a fixed end and persists it** (`finalize.go`, `fixerEndFixed` case). None. See F2.
17. **`gateAndFix` resume with `FixReview` pending and a passing gate runs the fixer** (the `passed && (rec.FixReview == "" || spent)` condition). None. See F2.
18. **Accepted record kept when the gate fails on resume, then the fixer changes the tree.** None. See F3.

**Findings**

- **F1 — LOW, design.** `fixaccept.go:375`: the branch that skips appending when the description already holds the heading, with `Accepted` still false, has no row. The retry-after-failed-write case therefore goes untested.
  - `TestAcceptFixReview` "a second accept writes nothing more" only exercises the `rec.Accepted` early return (`fixaccept.go:368`).
  - A missing description file (the read error) also has no row.
  - Evidence: `fixaccept_test.go:33-36`.
- **F2 — MEDIUM, design.** The crash-resume path with a pending review report is uncovered.
  - `finalize.go:629-658` has the `rec.FixReview` branch in `gateAndFix`.
  - `finalize.go:705-712` clears and persists `FixReview` after a fixed end.
  - No `finalize_gate_test.go` row seeds `FixReview != ""`; my grep found only `fixerattempts_test.go:21` setting it, a round-trip.
  - If either piece were broken, a resume would rerun the fixer on an already fixed tree, or skip the fixer a pending rejection needs, and no test would fail.
  - Fix: add a seeded-resume row for `FixReview` set with a passing gate and `Spent` below the cap, expecting a fixer session with `fix_review_path` set. Add one for a fixed end, asserting the on-disk `FixReview` is empty before the review runs.
- **F3 — LOW, design.** Nothing tests a record that is `Accepted` followed by a failing landing gate on resume.
  - `Accepted` persists in the record (`finalize.go:633-636`).
  - `judgeFixerChange` then waves any later fixer change through (`finalize.go:753`), including the one the new fixer turn makes.
  - The only accepted row seeds a passing gate (`finalize_gate_test.go`, "the operator's acceptance lands with no review").
  - This is the accepted-then-changed-tree case in the focus list.
  - Fix: add a row with `Accepted` set and a failing then passing gate, pinning the intended outcome, which is landing or a fresh review.
- **F4 — LOW, consistency.** The tapestry recipe has no assertion that Finalize declares no `on_reroute`, though darn and sampler do. Only tapestry's yaml lost the edge, with no test.
  - Fix: add an `OnReroute == ""` check for Finalize to the tapestry recipe test.
- **F5 — LOW, design.** The no-verdict halt path never asserts `rec.Findings`. `openFindings()` on a non-rejected review returns the `verdictSummary` line, which `AcceptFixReview` writes into the description.
  - The `wantReviews: 2` rows check only the halt reason.
  - Only the rejected path asserts `rec.Findings[0]`.
  - Evidence: `finalize_gate_test.go`, the "fix review's end decides" table.
  - Fix: assert `rec.Findings` has the `verdictSummary` line on one no-verdict row.
- **F6 — NIT, design (test economy).** The second half of "a spent cap halts naming the commits, and a resume… halts again" duplicates the seeded-resume row "a failing gate halts as a spent cap".
  - Both pin a resume over a failing gate with attempts spent and no fixer spawn.
  - The first-call halt and the `Halt == spent` assertion are new.
  - Fix: drop the resume half, or the duplicate seeded row.

**Tool calls**
1. One grep of `internal/loomrecipe/*_test.go` and `internal/shedrecipe/*_test.go` for `OnReroute` assertions, plus a grep for `FixReview:` in the landingshed tests. My context lacked the unchanged tapestry recipe tests and any test that seeds `FixReview`.
