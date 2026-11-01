```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I traced each changed function and branch with a concrete case.

**Per function and branch (case run, verdict)**

- `gateAndFix`, `Halt == review-capped` and not accepted: Halt cleared, rounds, `FixReview` and findings reset, record written. Correct.
- `gateAndFix`, `Halt == review-capped` and accepted: Halt cleared, `Accepted` kept. Correct for the plain resume. F1 below covers what happens if the tree changes afterwards.
- `gateAndFix` switch, passing gate with `FixReview` set and `Spent` below the cap: falls through to `runFixer`, so the fixer runs on the findings even though the gate passes. Intended and correct.
- `gateAndFix` switch, passing gate with `FixReview` set and `Spent` at the cap: goes to `judgeFixerChange`, which reviews afresh and clears `FixReview`. Correct.
- `judgeFixerChange` loop, cap 1, first review rejected: `ReviewRounds` reaches 1, so `haltReviewCapped` runs before the findings go to the fixer. This follows the entry ("at the cap Finalize escalates"). Correct.
- `judgeFixerChange` loop, cap 3, rejection then a fixed change within the caps: the nested `judgeFixerChange` proceeds without another review. Correct.
- `judgeFixerChange` loop, review returns no verdict below the cap: the record is written and the loop reviews again at the same HEAD. `ReviewRounds` rises on every pass, so it terminates. Correct.
- `judgeFixerChange` loop, `fixReviewEndChanged`: the record is written with the rounds and the halt is a `LandingStopPrefix` one. `Halt` is not `review-capped`, so a resume does not reset the rounds. Acceptable.
- `judgeFixerChange` loop, rejection followed by a `runFixer` spawn error: the record already holds `FixReview` and `Spent=0`. A resume with a passing gate takes `runFixer`'s path again. Correct.
- `judgeFixerChange` recursion through `runFixer` → fixed → `judgeFixerChange`: depth is bounded by `FinalizeFixReviewRounds`. Correct.
- `runFixer` fixed end: `FixReview` is cleared and written before the next judgement. Correct.
- `haltReviewCapped`: `Halt`, `Findings` and `Accepted=false` are persisted before the halt. The reason names `circling accept <slug>`. Correct.
- `endSpent`: reads the head, sets `Halt` to spent and halts. The reroute branch is gone. Correct.
- `AcceptFixReview`, retry after a failed write: if the description was appended but `Accepted` failed to write, the retry sees the heading and only sets `Accepted`. If `Accepted` is already true it returns nil, and the CLI then retries the commit. Idempotent and correct.
- `AcceptFixReview`, record of another entry or not review-capped: `ReadFixerAttempts` returns a zero record, so the result is `ErrNoFixReviewHalt` with nothing written. Correct.
- `finalizeCirclingVerb` and `rowEntry`: `rowEntry` is stable between the running and the halted history, so the entry number matches `readAttempts`. `LoomScratchDir` is the same `ScratchDir` the landing deps use (`landingdeps.go:101`). The `summaryparser` only checks the `# ` title, so the appended `##` section parses. Correct.
- `fixAcceptSection` with empty `Findings`: this cannot occur, because `openFindings` always returns at least one entry. Correct.
- `HandsBack`: Finalize now matches only `LandingStopPrefix`, and the removed reroute-cap clause cannot fire any more. Correct.
- `LoadConfig`: the new key is checked for a value below 1, and the template supplies the default. Correct.

**Findings**

- F1 — MEDIUM, design — `internal/landingshed/finalize.go` (`judgeFixerChange`, `if capReason == "" || rec.Reviewed == head || rec.Accepted`) and `fixerattempts.go` (`Accepted`).
  - The summary: `Accepted` is a bare bool that is not bound to the HEAD the operator accepted, unlike `Reviewed`. After `circling accept`, the next resume re-merges the parent and runs the gate.
  - A gate failure at that point spawns the fixer again (`Spent` is below the cap). `runFixer` → `judgeFixerChange` then skips the cap check because `Accepted` is still true, so a new over-cap change lands with no review and no findings in the description.
  - Operator commits made on the task branch between the accept and the resume land the same way.
  - The fix: record the accepted HEAD (for example `AcceptedHead`) and compare it to the current head, as `Reviewed` is compared. The record must carry that HEAD, because `AcceptFixReview` runs before the description commit moves HEAD, so either record the head after that commit or compare the Baseline..HEAD range.
  - Evidence: `finalize.go` (`judgeFixerChange` head, cap and `Accepted` check) and `fixaccept.go:381`.

- F2 — LOW, consistency — `internal/loomcli/circling.go` (`acceptFinalize`).
  - The summary: `AcceptFixReview` marks the record accepted and rewrites the description before the commit. If the commit fails, the dirty `summary.md` stays in the worktree and the record already says accepted. The retry path handles this, but a resume without a retry hits Finalize's clean-tree check.
  - That check halts, and the reason does not point at re-running `accept`. The fix: have the dirty-tree halt, or the accept refusal text, name "re-run `lym loom circling accept`".
  - Evidence: `circling.go` (`acceptFinalize`) and `fixaccept.go:376-382`.

**Tool calls**
1. `rowEntry` definition and `ScratchDir` in `landingdeps.go`: my context did not show the entry-index semantics or whether the circling scratch dir matches the landing one.
2. `summaryparser/summary.go` grep: to check that the parser tolerates the appended section.
3. A `ls` of `internal/summaryparser`, in the same call as the grep. It returned the file list and a grep warning because I guessed the wrong file name.
