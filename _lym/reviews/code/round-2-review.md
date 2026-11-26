---
verdict: BLOCKING
findings:
  - id: F8
    severity: BLOCKING
    class: design
    location: internal/driftengine/loosened.go:104
    summary: The range key hashes only the files ChangedFiles lists, which are the files that exist at the base, so a commit that adds or edits a test file created in the range never changes it. An uncaused ruling then stays reused and halts again, and a carried_by ruling naming a new file stays reused after that file drops the check.
    fix: "Make the range key cover every file the range adds as well as those it modifies or deletes. In internal/fabricengine add a method beside CommittedFileDiffs that returns the committed range's full diff including added files (ReadCommittedDiff already does this), wire it as a new seam in the drift deps (internal/driftengine/verify.go's deps and seam list, internal/loomcli/driftwiring.go), and have listFiles build rangeKey from that diff, not from rangeRulingKey(files). Keep the judge's own listed files as they are. Update the rangeRulingKey doc comment, the RangeKey comment in internal/driftengine/records.go, and the doc.go line \"It is reused only while every file diff of the range is unchanged\" to say the key covers every file the range adds, modifies or deletes. In internal/driftengine/verify_test.go, give the fixture the new seam and add a row to the `a loosened check in the review range` group: an uncaused ruling on limitsTest, then a second pass whose ChangedFiles is unchanged but whose full range diff adds a new test file holding the dropped checks; assert a second judge runs and the pass is Done. Extend the carried_by row the same way, with the carrier being an added file whose diff changes between the passes."
prior:
  - id: F1
    status: resolved
    answer: Step 2 now holds that a removed assertion another test holds, no looser, is not loosened, searched for across the repository and named under carried_by, and still rules a widened check or an uncarried removal loosened. Following it to the letter clears the halt once the judge runs again. The judge not running again when the carrier is a file the range adds is the range key's defect, raised as F8.
  - id: F2
    status: resolved
    answer: The new row keeps names_test.go ruled none across both passes, asserts the second request leaves it out, and compares its stored ruling whole with reflect.DeepEqual.
  - id: F3
    status: resolved
  - id: F4
    status: resolved
  - id: F5
    status: resolved
  - id: F6
    status: resolved
    answer: The first pass's range already holds limits_class_test.go, and the second pass changes only its diff; the second request lists limitsTest, which fails under a path-only key.
  - id: F7
    status: resolved
---

### [BLOCKING] The range key misses every file the range adds, so the re-judge does not fire when the dropped checks land in a new test file

**Location:** `internal/driftengine/loosened.go:104`

**Issue:**
`listFiles` builds `rangeKey` from `rangeRulingKey(files)`, where `files` comes from `d.ChangedFiles`.
`internal/loomcli/driftwiring.go:213` wires that to `fabric.CommittedFileDiffs`, and `internal/fabricengine/taskpatch.go:85-91` documents and implements that call as listing only files that exist at the base (`--diff-filter=MD`): "an added file is not listed".
So the range key changes only when a file that existed at the base changes.

That leaves the issue's own case open.
A deleted harness whose checks "moved into a test class" most often moves them into a class the range creates.
When the writer then commits the missing checks into that new class, no listed file's diff changes, the uncaused ruling's `Key` and `RangeKey` both match, the ruling is reused and Drift-Verify halts again with no judge.
The new way forward in the halt text ("or commit the dropped checks in a test") does not clear the halt in that case.
The amended entry's expected behaviour, re-judging "when a later commit in the range touches the test files the stored ruling compares against, such as the class a deleted check moved to", is not met.

The diff also opens a new hole through `carried_by`.
Suppose the judge rules a deletion `none` because a new test file of the range carries the check.
A later commit that drops the check from that new file leaves the range key unchanged, so the `none` ruling is reused and the loosening passes silently.
Before this diff the judge ruled the removal `loosened`.

The prose repeats the gap as a claim.
The doc.go line "It is reused only while every file diff of the range is unchanged", the RangeKey comment in `records.go` ("every file path and diff of the range"), `docs/overview.md` ("A change to a file of the review range has the judge rule on the uncaused file again") and `_lym/landing/summary.md` ("Any commit that changes a file of the range therefore has the judge rule on it again") all read as covering added files, and the code does not.
The tests cannot see the gap, since every carrier in the new rows (`limits_class_test.go`) is modelled as a file listed in `ChangedFiles`, that is, one that existed at the base.

## Focus answers

- Focus 1: the judge's Step 2, followed to the letter, rules a removal that another test carries out of the file's checks, names the carrier under `carried_by` and so clears the halt once the judge runs again. A widened check, or a removed one no test carries, still needs a trailer. The halt clears only when the answering commit changes a file that existed at the base (F8).
- Focus 2: the new row's `reflect.DeepEqual` on `names_test.go` fails if a ruling with no uncaused check stops being reused. Its second request listing `limitsTest` fails if the key stops covering a listed file's content. Both inputs of the key, path and diff, can change which ruling is reused. The key's missing input is the added files (F8).
- Focus 3: the halt text, the `checkAndJudge` comment and the `listFiles` comment match the code for files that existed at the base. The RangeKey comment, the doc.go reuse line, the overview line and the summary overstate the key's coverage (F8).
