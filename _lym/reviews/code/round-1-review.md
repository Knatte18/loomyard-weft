---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/websterengine/recordbatch.go:135
    origin: handler
    summary: The amend route of createTargetWayForward ends in recover-batch, which refuses as exhausted after two failed recoveries, because rebaseline records no amendment on a terminal-failed batch; the summary claims the opposite.
    fix: In failCardNotDone pass whether in.Batch.Recoveries has reached maxRecoveries to createTargetWayForward, and when it has, return the way forward RecoverSpawnOrAttach already gives for an exhausted batch (wayForwardSteps over resetVerb(ResetToBatchStart, number) and freshRestartSteps(stepRun)) instead of the amend-then-recover-batch text; keep amend-then-recover-batch for fewer recoveries. Delete the two sentences of _lym/landing/summary.md under "Way forward for a missing Create target" that say rebaseline --card makes the next recovery uncounted and that the amendment route works after both counted recoveries are used. Add a row for the exhausted case to the test in F2.
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/websterengine/recordbatch.go:222
    origin: lens:code-tests
    summary: No test reaches the signature-mismatch way forward, the cardNotDoneFor signature-mismatch flag, the uncheckable arm of createTargetWayForward, or the precedence between the delete, glyph and create way forwards.
    fix: Expose failCardNotDone and cardNotDoneFor through export_test.go in package websterengine and add one table-driven tier0 test in recordbatch_test.go with rows for a signature-mismatch finding from record-batch (amendment alone), an uncheckable record (fresh-restart steps), a create-not-done and glyph-rejected mix (glyph way forward wins), and a create-not-done and delete-not-done mix, each asserting the exact way forward clause. Drive the signature-mismatch row from a planindex.Finding with Check signature-mismatch through cardNotDoneFor so the flag is pinned too. No C# fixture is needed.
  - id: F3
    severity: LOW
    class: design
    location: internal/websterengine/recordbatch.go:139
    origin: lens:code-correctness
    summary: A signature-mismatch finding for a Rename pair's new side reaches a way forward that says amend the card's Create target, and no test row covers the Rename case.
    fix: In createTargetWayForward word the amend step as "the card's Create target or Rename new side", and add a row with checkID rename-not-done-new to TestDoneCheckVerdicts_NotFoundDetail in internal/planglyph/donecheck_test.go expecting the Rename pair's new side subject in the detail.
  - id: F4
    severity: LOW
    class: design
    location: internal/planglyph/donecheck.go:188
    origin: lens:code-tests
    summary: signatureMismatches has no negative rows, so nothing fails if it fires for a target without a parameter list, a different owner, a different unit or a Go plan.
    fix: Add rows to the table in TestDoneCheckVerdicts_NotFoundDetail in internal/planglyph/donecheck_test.go, each expecting a create-not-done finding that carries no signature-mismatch, for a C# target without parentheses, a declaration of the same name under another owner, a declaration of the same name under another unit, and a Go target.
  - id: F5
    severity: NIT
    class: consistency
    location: internal/planglyph/donecheck_test.go:222
    origin: lens:code-tests
    summary: TestDoneCheckVerdicts_NotFoundDetail checks the detail with strings.Contains although the detail is fully deterministic.
    fix: Compare findings[0].Detail to the expected string with equality, and write the not_found row's wantDetail as the whole detail string including the Create target prefix and the lookup text.
---

### [MEDIUM] Amend route dead-ends in an exhausted recover-batch

**Location:** `internal/websterengine/recordbatch.go:135`, `internal/websterengine/rebaseline.go:374`, `internal/websterengine/recoverbatch.go:509`
**Issue:** Issue #62 asks that a refused recovery never name a way forward that dead-ends.
`createTargetWayForward` ends its amend route with `lym webster recover-batch NN`.
`failBatch` marks the batch terminal, and `Rebaseline` skips the `AmendedCards` entry for a terminal batch (`if bs.Terminal { continue }`), so `rendersNewAmendment` is false when the next recover-batch runs.
`RecoverSpawnOrAttach` then refuses with `ErrRecoveryExhausted` once `prior.Recoveries` has reached `maxRecoveries`.
After the second counted recovery fails on the same create target, the refusal therefore names an amend route whose last step is refused, which is the dead end the issue describes one hop later.
`_lym/landing/summary.md` states that each `rebaseline --card` records an amendment that makes the next recovery spawn uncounted, so the route works after both counted recoveries are used; the code shows that claim holds only for an in-flight batch, not a failed one.

### [MEDIUM] Create way forward arms and precedence have no test

**Location:** `internal/websterengine/recordbatch.go:222`, `internal/websterengine/recordbatch.go:135`, `internal/websterengine/recordbatch.go:907`
**Issue:** `TestRecordBatch_DoneChecksBlockOnUnresolvedCreate` and `TestRecoverBatch_TerminalRunsTheSamePostBatchChecksAsRecordBatch` pin the create-not-done arms for record-batch and recover-batch.
Nothing fails if the `signature-mismatch` case in `cardNotDoneFor` is dropped, if `createTargetWayForward` ignores `signatureMismatch`, if its `uncheckable` arm changes, or if the `wayForward == ""` guards that order the delete, glyph and create way forwards are reordered.
Issue #61's corrective step lives only in the signature-mismatch arm.
The summary's claim that the recovery test covers the amendment-only text is true for the `recovered` arm only.

### [LOW] Rename new side told to amend a Create target

**Location:** `internal/websterengine/recordbatch.go:139`
**Issue:** `doneCheckVerdicts` runs the signature-mismatch check for both `create-not-done` and `rename-not-done-new`, and `signatureMismatchFinding` names the Rename pair's new side in the detail.
`cardNotDoneFor` sets `signatureMismatch` without telling the two apart, so the way forward says "amend the card's Create target" for a Rename card.
The Rename branch of `signatureMismatchFinding` has no test row.

### [LOW] signatureMismatches has no negative rows

**Location:** `internal/planglyph/donecheck.go:188`
**Issue:** The finding must fire only when the unit declares the member under the same owner and name with other parameter lists.
`TestDoneCheckVerdicts_NotFoundDetail` has one positive row.
A target without a parameter list, another owner, another unit and a Go plan are the guards that keep `create-not-done` for every other case, and none is pinned.

### [NIT] Contains where equality is available

**Location:** `internal/planglyph/donecheck_test.go:222`
**Issue:** `scribe:testing` prefers exact equality to a substring check.
The detail is built from fixed inputs and is fully deterministic.

## Rejected

- Master stencil wording (`contracts/stencils/webster/webster-template-master.md:165`, from lens:code-affected-set): the rung stops Master whenever the way forward names a plan edit, and the parenthetical is one example; the amendment is a plan edit, and the glyph-rejected way forward set the same precedent without a stencil change.
- The `recovered` flag derived from `in.Verb == "recover-batch"` loses the recover-batch hint on a first create-not-done found during a recovery (lens:code-correctness): the amend route still makes progress, and a recovery that fails the same target is exactly the case issue #62 names.
- The unit file count repeats once per target, against issue #63's once-per-unit request (handler): the file list is gone and what repeats is one short count, so the multi-kilobyte repetition the issue reports no longer exists.
- Zero-unit-file branch of `searchedDescription` unpinned (lens:code-tests): the lens did not confirm it is uncovered, and the "unit missing" row of `TestDoneCheckVerdicts_Rules` reaches a not_found without unit declarations.
- `sharedUnitSegments` splitting a Go unit at `.` (lens:code-correctness): it only affects ordering and is harmless.
- `.`-segment splitting for C# units unpinned (lens:code-tests): `sharedUnitSegments` is a helper the public `doneCheckVerdicts` test already reaches through the Go ordering row, and a per-helper test is test-economy noise.
