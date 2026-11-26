```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the review's proposals that the fixer carried out (F1's extended version included), looking for cases where a proposal is wrong or breaks something elsewhere.

1. **LOW**: `_lym/landing/summary.md`, title and "Design decisions" (F5 proposal).
   - The proposal corrected only the "What changes" sentence, so the same overclaim stands elsewhere in the file.
   - The title still reads "re-judges an uncaused loosened ruling after a new commit".
   - Lines 20-22 still read "any new commit is the writer's attempt to answer it, so one judge spawn per new commit is the right cost".
   - The corrected line now says that only a commit changing a file of the range re-judges. A message-only commit with no trailer, or a change plus its revert, spawns no judge.

2. **LOW**: `internal/driftengine/verify.go`, `checkAndJudge` doc comment, "is a Stuck, with no judge, while the range is unchanged" (F4 proposal, combined with F7).
   - Concrete case: a trailer-only commit (`TestDriftVerify`, "a trailer-only commit passes").
   - No file diff changes, and after F7 `rangeRulingKey` is identical. `records.go` and `rangeRulingKey` define the range key as paths and diffs only.
   - A judge is still spawned, because `fileRulingKey` hashes the trailers.
   - So "range unchanged" does not imply "no judge". Wording such as "while the range's files and trailer lines are unchanged" would hold.

3. **LOW**: `contracts/stencils/drift/drift-template-judge.md`, Step 2 (F1).
   - The sentence just above the new paragraph still says "Decide from the file's diff what is a test and what is loosened".
   - The new paragraph says a removed check's status depends on other files, including unlisted ones the judge must search for.
   - A judge following the first sentence would rule a moved check `loosened` from the deleted file's diff alone.

4. **LOW**: `drift-template-judge.md`, the new moved-check rule (F1), against its siblings.
   - `contracts/stencils/loom/loom-rubric-code-review.md:85` still makes any assertion the diff "widens or removes" in a pre-existing test a finding unless a `Loosened-Check:` trailer names it.
   - `loom-fragment-loosened-check.md` likewise requires a trailer for every removal.
   - A writer who takes the halt's "commit the dropped checks in a test" way forward passes Drift with no trailer. Code-Review, which runs next on the darn rows, would then raise the same moved check as a finding.
   - This is a policy split the parent should either accept knowingly or close.

I found nothing wrong with the F2, F3, F6 or F7 proposals. The F7 premise holds, since reuse also requires `held.Key == key`, and that key hashes the trailers.

Tool calls: none.
