```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the fix's change set agrees with its surroundings.

1. **MEDIUM**: the new moved-check rule in `drift-template-judge.md` Step 2, with the matching sentence in `doc.go`, disagrees with the reuse rule.
   - A deleted file's `none` ruling can now rest on a test in another file.
   - That ruling is still reused by its own file key (diff and trailers), a reuse the F2 row now pins.
   - If a later commit drops the check from the carrying file, the deleted file is not listed again.
   - If the carrying file is new in the range, its diff against the base shows no removed assertion, so the check can vanish with no halt.
   - Evidence: `listFiles` applies the range key only when `holdsUncausedCheck(held.Checks)`, and `records.go` `JudgedFile.Key` says the ruling is reused while the diff and messages are unchanged.

2. **MEDIUM**: the same moved-check rule disagrees with two siblings that were left unchanged.
   - `contracts/stencils/loom/loom-rubric-code-review.md:85` still says an assertion the diff "widens or removes" is a finding unless a `Loosened-Check:` trailer names it.
   - `loom-fragment-loosened-check.md` tells every committer to record each removal as a trailer.
   - So following the halt's "commit the dropped checks in a test" clears Drift, but Code-Review or Drift-Recheck's neighbouring perch can still raise the moved check as untrailered.

3. **LOW**: `internal/driftengine/verify.go:76` is the twin of F4 and was left as it was.
   - It says "the judge rules on the loosened checks of every file with no stored ruling under its key".
   - An uncaused ruling stored under an unchanged file key is now judged again when the range changes.

4. **LOW**: `_lym/landing/summary.md` "Design decisions" and the title keep the wording F5 removed elsewhere in the same file.
   - "any new commit is the writer's attempt to answer it, so one judge spawn per new commit is the right cost".
   - The title says "after a new commit".
   - "What changes" now says only a commit that changes a file of the range triggers the re-judge.

5. **LOW**: in `drift-template-judge.md` Step 2, the unchanged sentence just above the new paragraph still says "Decide from the file's diff what is a test and what is loosened". The new text tells the judge to search the repository's tests before ruling a removal.

6. **LOW**: `docs/overview.md:1121`, next to the rewritten line 1122, defines a loosened check as one "widened in a test". It mentions neither removal nor the new exception for a check another test holds, while the line below now speaks of "dropped checks".

Tool calls:
- One `grep` for statements about reuse and loosening across `driftengine`, `driftcli`, `contracts` and `docs`: my context held only the files the findings named, not the sibling statements.
