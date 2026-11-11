---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    origin: lens:code-tests
    location: internal/loomcli/decisionreplace_test.go:132 (TestDecisionReplaceVerb_Refusals)
    summary: two refusals decisionReplaceVerb adds have no test that reaches them, against PATTERN-refusal-way-forward and refusal-spec
    fix: In TestDecisionReplaceVerb_Refusals add two rows. One sets the fake so deps.rowWritten reports true and the record is uncommitted or differs from its committed copy, asserting the checkDarnRecord refusal text and its way forward. One makes the record file absent (fs.ErrNotExist from readRecord), asserting the "has no decision record at" text and decisionWayNoRecord. Each row keeps the existing record, HEAD and board unchanged assertions. Add also a row in TestDecisionReplaceVerb_CommitOutcomes for the landReplace refusals that are not a raw I/O failure propagated with context  the unreadable-HEAD refusal, the headCarries error and the failed restore  each asserting its message and way forward.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/loomcli/decisionreplace_test.go:562
    summary: the AddOnABeamRun row of TestDecisionCalls_RefreshEveryStoredRecord duplicates the no-stored-record row and never reaches the beam exclusion
    fix: Make the row exercise the exclusion D3 asks for. Either build the decisionFake so boardRecordStored runs the real record.project check with a stored record present on the entry and assert f.stored stays empty after add, or delete the row and cover the record.project branch of boardRecordStored in a test that drives that closure with a beam project record.
  - id: F3
    severity: MEDIUM
    class: consistency
    origin: lens:code-affected-set
    location: internal/loomcli/decision.go:1875
    summary: the decision store help text still says store refuses a recipe with no Discussion approval and says nothing of the filed raddle refusal
    fix: Rewrite the Long text of the store command. It copies the committed record onto the entry as add and replace do; it is admitted past the Discussion approval, or on a recipe with no approval row when the entry carries a stored record; it refuses a run still inside Discussion-Review, a record HEAD does not hold, an entry that has landed, and a raddle run whose filing is done.
  - id: F4
    severity: MEDIUM
    class: consistency
    origin: handler
    location: contracts/specs/loom-discussion-spec.md:132
    summary: the D6 review criterion asks the discussion spec to name replace and --record-file, and the spec names only replace
    fix: In the "Parts no writer authors" bullet group, after the replace bullet, add one sentence saying a Sketch's stored record, which holds added entries once a motif run has made it, is folded with `lym board upsert --record-file`, so the spec names both verbs and says when each is used.
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/discussionparser/replacecheck.go:240
    summary: CheckAddedEntriesSubset tests set membership, so a file that repeats an existing added entry passes, though D1 says the file adds none and compares byte for byte
    fix: Compare as a multiset or an ordered subsequence. Count each current entry's text, and report a next entry once its count is used up. Compare the entry text without trimming trailing whitespace, or state in the doc comment that the trim applies. Add a duplicate-entry row to TestCheckAddedEntriesSubset.
  - id: F6
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: docs/overview.md:814
    summary: three beam project-record texts still name decision add as the only way the scope changes, while the declared PATTERN-run-records text and the verb admit replace
    fix: In docs/overview.md:814, contracts/stencils/loom/loom-directive-project-record.md:20 and contracts/recipes/pawl-recipe.yaml:15, name `lym loom decision add` or `replace` where each says add is the only writer.
  - id: F7
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/loomcli/decisionreplace_test.go:447 (TestDecisionLock_SerializesRecordWriters)
    summary: decisionStoreVerb now takes the decision lock and no row asserts it, nor a lock failure's refusal
    fix: Add a store row to the lock test or a row to a store test asserting lockCalls equals lockReleases after a store, and a lockErr row asserting a refusal carrying decisionStoreRetry.
  - id: F8
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/loomcli/recordhistory_test.go:114
    summary: TestRecordHistoryCut re-implements the recordHistory glue inline, so the fallback to the whole history when no cut is found is never reached through recordHistory
    fix: Extract the trailer lookup and cut from the closure body in recordHistory into one function taking the trailers and versions, call it from the closure and from the test, and add a row whose cut is not found asserting the whole history is returned.
---

### [BLOCKING] Two replace refusals have no reaching test
**Location:** internal/loomcli/decisionreplace_test.go:132
**Issue:** `PATTERN-refusal-way-forward` and `contracts/specs/refusal-spec.md` require every refusal a verb can return to have a test that reaches it and asserts its message, way forward included, unless it is a propagated raw I/O failure or a wiring guard. `decisionReplaceVerb` adds the Record-Write run refusal from `checkDarnRecord` and the absent-record refusal; the table has no row for either. `landReplace` adds refusals for an unreadable HEAD, a `headCarries` error and a failed restore, and `CommitOutcomes` reaches none of them.

### [MEDIUM] The beam-run row of the refresh test is vacuous
**Location:** internal/loomcli/decisionreplace_test.go:562
**Issue:** The row sets `boardStored` to its default, so it repeats the no-stored-record row. The `record.project` exclusion in `boardRecordStored` is reached by no test, so the D3 criterion that `add` on a beam run leaves the project entry's stored record unchanged has no row that would fail if it broke.

### [MEDIUM] The store help text contradicts the verb
**Location:** internal/loomcli/decision.go:1875
**Issue:** The help says store refuses a recipe with no Discussion approval and gives no filed-raddle refusal. `decisionStoreVerb` now admits a no-approval run whose entry carries a stored record and refuses a raddle run whose filing is done.

### [MEDIUM] The discussion spec does not name `--record-file`
**Location:** contracts/specs/loom-discussion-spec.md:132
**Issue:** The review criterion for D6 asks the discussion spec, the parent directive and the overview to name `replace` and `--record-file`. The spec bullet names `replace`, when to use it rather than `add` and the history reset, and does not name `--record-file`.

### [LOW] The added-entry subset check accepts a repeated entry
**Location:** internal/discussionparser/replacecheck.go:240
**Issue:** Membership in a set of entry texts lets a file hold an existing added entry twice, which adds an entry. Entries are also compared after a trailing-whitespace trim, short of "byte for byte".

### [LOW] Beam project-record texts still name add as the only writer
**Location:** docs/overview.md:814
**Issue:** The overview line, the project-record directive stencil and the pawl recipe comment say only `decision add` changes a beam project's scope. `replace` is admitted on beam runs and the declared `PATTERN-run-records` text names it.

### [LOW] The store verb's lock is unpinned
**Location:** internal/loomcli/decisionreplace_test.go:447
**Issue:** The lock test covers `add` and the carry-over writer. `decisionStoreVerb` takes the lock and no row asserts that or the refusal when the lock fails. The real closures `headCarriesReplace`, `raddleBoardFiling` and `raddleEntryTask` are reached only through fakes.

### [LOW] The history fallback is not reached through recordHistory
**Location:** internal/loomcli/recordhistory_test.go:114
**Issue:** The test rebuilds the lookup and cut inline, so a change to the closure in `recordHistory` leaves it green. The integration test never reaches the path where the trailer's sum matches no version.

## Rejected

- Lens code-correctness finding 2 (index may keep the written bytes staged after a failed commit): unconfirmed by the lens, the same exposure `add` has today, and a path-scoped commit takes the working copy.
- Lens code-correctness finding 3 (full first-parent walks on every `recordHistory` call): a cost note with no incorrect result.
- Lens code-correctness remark on `parseRecordReplaced` splitting a path with a space: unreachable, the record path is slug-built under `_lym`.
- Lens code-affected-set finding 3 (`docs/overview.md:1160` lists `--by parent|operator` for `add`): the lens itself marked it as predating the diff.
- Lens code-tests finding 6 items on uppercase base hashes, `SkipGit` and merge or root commits in the history seam: each is a one-line branch covered in effect by the existing rows or by the exact-equality assertions, and they add no behaviour the answer key names.
- Lens code-tests findings 3 and 4 as separate items: merged into F1.
- Lens code-tests findings 2 and 5 as separate items: merged into F7.
- Comments in the diff that name other symbols, such as the `decisionReplaceVerb` doc naming `decisionVerb`: the surrounding code does the same throughout, so it is the repository's convention.
