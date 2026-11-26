---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-affected-set
    location: contracts/stencils/drift/drift-template-judge.md#Step 2
    summary: The new third way forward, committing the dropped checks in a test, cannot clear the halt, because the judge's file rules still force the deleted file loosened and each dropped check uncaused without a Loosened-Check trailer.
    fix: "In Step 2's file rules of contracts/stencils/drift/drift-template-judge.md, tell the judge that an assertion the diff removes from one file but another file of the request's `files` adds back, the same check on the same behaviour, is not loosened: rule it `none` for that file, or leave it out of the file's checks, and name the file that carries it in the reason. Keep the rule that a widened or dropped check with no such counterpart needs a trailer. Then make the fake judge in the new TestDriftVerify row of internal/driftengine/verify_test.go stand for that rule only if the fixture models it; no prose assertion on the stencil."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/driftengine/verify_test.go:1343
    summary: No test pins that a stored ruling with no uncaused check is still reused by its file key when another file of the range changes; the `!holdsUncausedCheck(held.Checks) ||` clause at loosened.go:108 can be dropped with every test passing.
    fix: "Add a row to the `a loosened check in the review range` group of TestDriftVerify in internal/driftengine/verify_test.go, or extend the new row: keep a second file ruled `none` (or `caused`) in the range across both passes, change another file between the passes, and assert the second judge request does not list the `none` file and its stored ruling (Key, Strand, At) is unchanged."
  - id: F3
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: docs/overview.md:1122
    summary: The overview still says an uncaused check is Stuck until a Loosened-Check trailer commit or a revert, leaving out the new way forward and the re-judge on a changed range.
    fix: "Rewrite docs/overview.md:1122 to name the three ways forward the halt now names (a Loosened-Check trailer commit, a revert, or a commit adding the dropped checks to a test), then `lym loom resume`, and that a change to the range has the judge rule on the uncaused file again."
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/driftengine/verify.go:257
    summary: checkAndJudge's doc comment still says a stored uncaused ruling is a Stuck with no judge, which now holds only while the range is unchanged.
    fix: "Change the line at internal/driftengine/verify.go:257 to: `// A stored ruling that a loosened check has no measured cause is a Stuck, with no judge, while the range is unchanged.`"
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/driftengine/loosened.go:191
    summary: The halt text and comments claim any new commit, or a resume, has the judge rule on the range's files again, but the range key hashes only file diffs and trailer lines, and only the uncaused or changed files are listed.
    fix: "In internal/driftengine/loosened.go:191 replace `which has the judge rule on the range's files again` with `which has the judge rule again on a file whose stored ruling the new commits may answer` (keep the test's requireStuck substring in verify_test.go:1354 in step). Replace the uncausedReason doc line at loosened.go:185 with `// A commit that changes a file of the range has the judge rule on the uncaused files again.` In internal/driftengine/records.go:176 replace `so a later commit is judged again` with `so a later change to any file of the range is judged again`. In _lym/landing/summary.md replace `Any new commit in the range therefore has the judge rule on it again.` with `Any commit that changes a file of the range therefore has the judge rule on it again.`"
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/driftengine/verify_test.go:1374
    summary: The new row only adds a new path to the range, so a range key built from paths alone would pass; no test pins that a content change to a file already in the range re-judges a stored uncaused ruling on another file.
    fix: "In the new TestDriftVerify row in internal/driftengine/verify_test.go, have the first pass's range already hold the second test file with a first diff, and change only that file's diff in the second pass; assert the second judge lists both files as now."
  - id: F7
    severity: NIT
    class: design
    origin: handler
    location: internal/driftengine/loosened.go:55
    summary: The trailer lines in rangeRulingKey can never change which ruling is reused, since the reuse check also requires the file key, which already hashes them; this is the mutation survivor no test can kill.
    fix: "Drop the `trailerLines` parameter and the `hash.Write` of the trailers from rangeRulingKey in internal/driftengine/loosened.go, pass only `files` at loosened.go:104, and remove `and the trailer lines` from its doc comment and `and trailer line` from the RangeKey comment in internal/driftengine/records.go:175."
---

### [BLOCKING] The new way forward cannot clear an uncaused ruling under the judge's instructions

**Location:** `contracts/stencils/drift/drift-template-judge.md`, Step 2, the file rules; promised at `internal/driftengine/loosened.go:191` and `internal/driftengine/doc.go:241`.

**Issue:** The diff now re-judges a stored uncaused ruling once the range changes, and the halt advertises "commit the dropped checks in a test" as a way forward.
The judge is told to rule a file `none` only when "the diff widens or removes no assertion, tolerance or limit in a test", and a check `caused` only when a `Loosened-Check:` trailer names it, `uncaused` otherwise.
A deleted harness whose checks moved into a class still removes assertions in its own diff, and the commit that adds them carries no trailer.
So the re-judge the diff triggers, followed to the letter, rules the deleted file `uncaused` again: exactly the halt from issue #249, now with one more judge spawn.
The new test passes only because its fake judge returns no file rulings (`f.fileRulings = nil`).
The halt names a way forward that does not lead forward, against `PATTERN-refusal-way-forward`, and the board entry's "Expected" (re-judge the loosened file once the code answered it) is met in mechanism only.

### [MEDIUM] Reuse of a ruling with no uncaused check across a range change is unpinned

**Location:** `internal/driftengine/verify_test.go:1343`

**Issue:** The rows at `verify_test.go:1282` and `:1449` reuse a caused ruling only over an unchanged range, and the new row holds only an uncaused ruling.
Dropping `!holdsUncausedCheck(held.Checks) ||` at `loosened.go:108` re-judges every file on every new commit with no test failing, though the change description states this behaviour as a design decision.

### [LOW] The overview names only two ways out of an uncaused ruling

**Location:** `docs/overview.md:1122`

**Issue:** The line reads "until a `Loosened-Check` trailer commit or a revert, then `lym loom resume`"; `:1371` inherits it for Drift-Recheck.
The diff adds a third way forward and the re-judge on a changed range, so the doc no longer agrees with the halt.

### [LOW] checkAndJudge's doc comment states the old reuse rule

**Location:** `internal/driftengine/verify.go:257`

**Issue:** "A stored ruling that a loosened check has no measured cause is a Stuck, with no judge" no longer holds once the range changes.

### [LOW] The wording overclaims what triggers a re-judge and what is judged

**Location:** `internal/driftengine/loosened.go:191`, with `:185`, `internal/driftengine/records.go:176` and `_lym/landing/summary.md` "What changes".

**Issue:** The range key hashes file paths, diffs and trailer lines (`loosened.go:49-57`).
A commit that leaves those unchanged, such as a message-only commit with no trailer or a change and its revert, re-judges nothing, yet the comments and the change description say any new commit does.
The halt says resume "has the judge rule on the range's files again", but only uncaused or changed files are listed.

### [LOW] No test pins that the range key covers file content, not only paths

**Location:** `internal/driftengine/verify_test.go:1374`

**Issue:** The second pass adds a new path, so a key over paths alone passes the row.
The issue's case, checks added to a class the range already changed, is a content change to a file already listed.

### [NIT] The range key's trailer input is redundant

**Location:** `internal/driftengine/loosened.go:55`

**Issue:** `listFiles` reuses a ruling only when `held.Key == key`, and `fileRulingKey` already hashes the trailers, so a trailer change lists the file whatever the range key holds.
That is why the mutation report's survivor at this line cannot be killed by any test; the input and its doc mentions can go.

## Focus verdicts

- Re-judge inputs: the judge sees the uncaused file and every changed or new file, since an answering commit changes the receiving test's diff; under the current stencil it cannot rule the moved checks anything but `uncaused` without a trailer (F1).
- Callers: `checkAndJudge` and `passCommand` both call `listFiles` before ruling on reuse; the early return at `verify.go:168` fires only after a pass, which an uncaused halt never records. Not a defect.
- Wording: F5.
- Docs outside the package: F3; `contracts/orchrole/orch-role.md` names no ways forward and stays correct.
- Merge danger: `range_key` is `omitempty`, decoded by plain `json.Unmarshal`, and an older binary that drops it costs one extra judge spawn. The `two-way` call and the `drift` blast radius hold.
- Mutation survivor: F7; reuse of a non-uncaused ruling across a range change is unpinned (F2).

## Rejected

- Root cause: the `Root-Cause:` trailer on `e6393683f` names the defect the diff removes, reuse by the file's own key and trailers alone; no finding.
- Scope: the trigger on any range change is wider than "the test files the ruling compares against", but the entry's "such as" and the ruling's lack of a compared-file list license it; no finding.
