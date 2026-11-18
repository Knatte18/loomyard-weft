```yaml
lens: interactions
outcome: reported
```

Task: check the record's decisions in pairs (interactions lens) for one undoing, outrunning or contradicting another.

**Findings**

1. **BLOCKING — D3 with D1 and Parent-Align (diff base).** D3 diffs from the drift baseline commit, but D1 and the trailer search use the review range, so tests the parent loosened count against the task.
   - D3 lists files that exist at the run's *baseline commit* (the branch point) and that the committed code changes. Its causes come from the *review range's* commit messages.
   - In tapestry, Parent-Align merges the parent tip before Drift-Verify. Any test the parent itself loosened then shows up as loosened against the branch point.
   - The cause would be on the parent's commits, which are outside the review range. The rubric says "the parent's own commits the merge brings in are no subject".
   - Result: the judge rules `uncaused` and the run halts as Stuck for something the task never did.
   - D3 needs to name the same base the review range uses, or exclude changes the parent brought in.

2. **MEDIUM — D1 against D3 (which causes count).** D3 accepts a parent amendment through `lym drift amend` as a cause, but D1 accepts only a `Loosened-Check:` trailer.
   - A loosening the parent excused passes Drift-Verify, and then Code-Review files it as a finding.
   - The fixer cannot add a trailer to the commit that loosened the check without rewriting history. Its only moves are to revert the loosening, which undoes what the parent accepted, or to fight the finding.
   - The two decisions disagree on what counts as a stated cause.

3. **MEDIUM — D1 and D2 (fixability).** D1 requires the trailer "on the commit that loosens it". When the writer forgot it, no fix in the Code-Review ring can satisfy D1 short of rewriting history or reverting and re-committing. D1 should accept a trailer anywhere in the range, or the record should name the fix path.

4. **MEDIUM — D6 against the rubric's existing Amendment and Review input sections.** Today a review input reaches the reviewer only through the amendment note. The rubric also says "when the amendment note is not `none` … review only the change it names". D6 needs a whole-range review plus the PR findings. If the PR findings ride the amendment marker, they narrow the review to the amendment and defeat D6's purpose of finding the misses' siblings. The record has to say that the PR findings input does not set the amendment, or change that rubric rule.

5. **MEDIUM — D3 against its own acceptance criterion.**
   - The criterion refuses "a judgement whose `loosened` list leaves out a listed file's ruling". But D3 defines `loosened` entries per loosened check, not per file, and a listed file with no loosened check has no entry.
   - Go lists every changed file that existed at the baseline, tests or not, because no test pattern is hard-coded. So nearly every changed file has no entry, and validation would refuse every honest judgement.
   - The same gap breaks the "stored ruling is reused" criterion, because a file with no rulings stores nothing to reuse, so Drift-Recheck re-spawns the judge every time.
   - D3 needs a per-file "no loosened check" ruling, or the criterion has to change.

6. **MEDIUM — D4 with D6 (friction note path on a repeated send-back).** The reroute cap allows up to 3 send-backs. The friction writer keys a note by row, like `_lym/loom/friction/&lt;Row&gt;.md`. A second send-back from PR-Review overwrites the first miss's note, or collides with a note the PR-Review agent writes itself, so the first miss stops being visible. The same applies to the stored findings file under `_lym/reviews/code/` if it has a fixed name.

7. **LOW — D2 with D4 (fix-here path).** The PR-Review agent's fix-here commits come after Drift-Recheck and Code-Review. A loosened check it commits gets only D2's trailer rule, and no row checks it. That bound should be stated in D4.

8. **LOW — D4 with D5 (what counts).** D4 counts "the findings" against `pr_review_small_max`, and the burler parser also knows NIT. The record does not say whether LOW and NIT findings count.

9. **LOW — D3 in `diff: command` mode.** D3 puts the list "beside the changed cells". Command mode has `cases`, not cells, and the record does not say whether loosened checks are ruled in that mode too.

**Pairs checked**

- D1 × D2, trailer as the cause: holds.
- D1 × D3, cause sources: broken (2).
- D1 × D2, fixing a missing trailer: broken (3).
- D3 × Parent-Align and the D1 range base: broken (1).
- D3 × its acceptance criteria (per-file validation and reuse): broken (5).
- D3 × Drift-Recheck after a D4 send-back: holds.
- D3 × the `drift-judge-and-line-range` peer: holds; it is already an open risk.
- D4 × D5, threshold and budget-0 fallback: holds, apart from (8).
- D4 × D6, order of store, note and stop: holds, since the stop comes last.
- D4 × D6, repeated send-back: broken (6).
- D6 × the rubric's Amendment and Review input: broken (4).
- D4 × the reroute edge's fresh episode, Code-Bouncer `carry_over` and Describe's `budget.Code-Review` gate: holds.
- D4 × `pr_review_ask_parent` on re-entry: holds.
- D7 × D1, figures in the description at Code-Review and after Describe: holds.
- D2 × D4, fix-here commits: gap (7).
- D3 × `diff: command`: unspecified (9).
- Constraints (fabric boundary, sole parsers, tolerant records, rows-not-name) against D3 and D4: hold.

I made no further tool calls.
