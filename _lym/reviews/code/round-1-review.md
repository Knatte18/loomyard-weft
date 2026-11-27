---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: decision
    location: pattern/PATTERN-test-only-code.md:15
    summary: "Functions only tests call sit in production files, which PATTERN-test-only-code forbids; the added decision allows them, but the PATTERN text does not and the task declares no change to it"
    fix: "Keep the code and the allowlist entries, which the added entry 'A chain part may leave production code that only a named later part first calls' requires, and defer to the run's parent to rule how the constraint and that entry are reconciled. If the parent rules the PATTERN text is to carry the allowance, add one bullet under '## Rule' of pattern/PATTERN-test-only-code.md saying what the new doc comment of testOnlyCodeAllowed in cmd/lym/testonlycode_linux_test.go says: a function of a fabric-core chain part that only tests reach so far may stay in its production file, its allowlist reason names the later part that adds the first production caller, and that part removes the entry with the caller."
    origin: lens:code-affected-set
  - id: F2
    severity: MEDIUM
    class: scope
    location: internal/fabricengine/executor_test.go#TestRunRule
    summary: No row gives a re-read a transport failure, so recording it and asking the rule again is unchecked
    fix: "Add to TestRunRule a row whose action list re-reads FieldOriginRefs after a PhaseOrigin request, whose second read call returns a *fieldReadError wrapping the provoked transport error, and which wants a nil error and one ReadFailure with Phase PhaseOrigin. Merge it into an existing row where possible, and keep the part's fabric _test.go total below the base total of 58594 lines."
    origin: lens:code-tests
  - id: F3
    severity: MEDIUM
    class: scope
    location: internal/fabricengine/executor_test.go#TestRunRule
    summary: "No row has a joined read error that mixes a transport failure with another failure, so the all-or-nothing rule that keeps a real read error from being recorded as an unreachable origin is unchecked"
    fix: "Add to TestRunRule a row whose phase read returns errors.Join of a *fieldReadError wrapping the provoked transport error and a *fieldReadError wrapping errors.New, and which wants that joined error returned and no ReadFailures."
    origin: lens:code-tests
  - id: F4
    severity: MEDIUM
    class: scope
    location: internal/fabricengine/snapshotread_integration_test.go#TestSnapshotReader_ReadsAHub
    summary: "No test checks that one failing store stops no other: the remaining stores are filled and the error joins one fieldReadError per failing store in walk order"
    fix: "Add a case to TestSnapshotReader_ReadsAHub that reads FieldLocalRefs (or FieldOriginRefs) with StoreCode and StoreBolt pointing at directories that fail and StoreRecords at a good one, and assert that the records refs are filled, that the failed stores have no entry, and that the error's Unwrap() []error parts name the code store and then the bolt store (assert on the message prefixes 'read local_refs of the code store: ' and 'read local_refs of the bolt store: ')."
    origin: lens:code-tests
  - id: F5
    severity: MEDIUM
    class: scope
    location: internal/fabricengine/snapshotread_integration_test.go#TestSnapshotReader_ReadsAHub
    summary: "FieldRecordsStore is only read for an identity layout and a broken one, so dropping Heads and Tags from the result, or breaking the absent-directory answer, fails no test"
    fix: "In the RecordsStoreWhoseLayoutCannotBeRead case, before the refspec is unset, read FieldRecordsStore from the one-repository hub and want Present true with Heads and Tags equal to the literal records namespace prefixes under refs/lym/ (call ForgetOriginNamespaceForTest before the broken read as now). Add one read with StoreRecords pointing at a path that does not exist and want the zero RecordsStore and a nil error."
    origin: lens:code-tests
  - id: F6
    severity: MEDIUM
    class: decision
    location: internal/fabricengine
    summary: "The part's fabric test code shrinks by 394 of 58594 lines (0.67 percent), and the added entry asks review to raise a part whose test code barely shrinks"
    fix: "Leave the code as it is and defer to the run's parent, who rules whether this net shrink satisfies the added entry 'The restructure cuts fabric's test code, it does not only move it' or which further fabric tests are to be cut."
    origin: lens:code-scope
  - id: F7
    severity: MEDIUM
    class: consistency
    location: internal/fabricengine/destroy_test.go#TestGate_Containment
    summary: Tests that existed at the base are deleted with no Loosened-Check trailer although no test in another file holds their assertions
    fix: "Add a commit to the branch (it may be the fixer's own commit) with one 'Loosened-Check: <test> — <change> — <cause>' line, the cause naming what was measured or read, for each of TestMergeContinue_FreshHandle_RecoversCrashedAfterCleanStaging, TestDropHandOverMark, TestCommit_TwoSided_WarpFirstOrdering, TestGate_CheckOrdering, TestGate_DirtinessNAEmptyReason, TestNew_HappyPath, TestHubLogsDir_MkdirAllIdempotentAgainstFabricCreatedDotLym and TestOriginRecordPaths. Then go through the 'Parts of tests deleted' list in the body of commit f4fd75355 and add the same line for each trimmed test whose removed assertion is held by no test in another file."
    origin: handler
  - id: F8
    severity: LOW
    class: consistency
    location: internal/fabriclayout/mutation.go:16
    summary: Doc comments moved into the layout still describe their values by engine-private functions, an engine file and engine error types
    fix: "In internal/fabriclayout/mutation.go drop '(destroy.go)' and rewrite the Kind comments that name removePath, removeGitWorktree, removeLink, deleteBranch, deleteRemoteBranch, updateRemoteBranch, createExclusiveDir, createGitWorktree, archiveWeftTip and setAsideTag so each says what the kind records (for example 'a deletion of a single path or a directory tree'). In internal/fabriclayout/verbresults.go reword the comments at lines 295 and 506 so they do not name WeftStopError or ErrStrayPathKept."
    origin: lens:code-affected-set
  - id: F9
    severity: LOW
    class: scope
    location: internal/fabricengine/executor_test.go#TestRunRule
    summary: "Smaller branches of the loop and the reader have no failing test: a phase read twice successfully, a nil lock, a tag naming no commit, and a prunable records worktree in the anchors read"
    fix: "Fold these into existing rows: let one TestRunRule row request the same phase twice with both reads succeeding and want it once in Phases, and pass a nil lock in one row; in TestSnapshotReader_RefStorage add one tag on a blob (git tag on the object name of a.txt) to each repository, with the wanted refs unchanged; in TestSnapshotReader_ReadsAHub remove one records worktree directory before an anchors read and want no entry and no error."
    origin: lens:code-tests
---

The review range is `6fbe0ff7a..HEAD`.
The moves themselves read as renames: the serialized field tags and marshal methods match the base as a set, the engine declares no alias of a moved name, neither new package calls a disk-reading `path/filepath` function, and both declared PATTERN changes are landed.
I did not build or run any test; the verify gate owns that.
The diff pieces for most retargeted packages were not read line by line: I checked them with a comparison that strips the `fabricengine.` and `fabriclayout.` qualifiers and read the added production lines it left.

### [BLOCKING] Test-only functions in production files against PATTERN-test-only-code

**Location:** `pattern/PATTERN-test-only-code.md:15`, `cmd/lym/testonlycode_linux_test.go:388`

**Issue:** `runRule`, `outcomeClassOf`, `fieldReadError.Error`, `fieldReadError.Unwrap`, `snapshotReader.read`, `readLocalRefs` and `Refusal.Error` are reached by tests alone and live in production files.
The entry says such a function is never in a production file, and its background admits only shared helpers no kit can hold or functions the analysis cannot see.
The record's added entry of 2026-11-27 allows exactly these, so the constraint and the answer key conflict.
The round's rules say the constraint wins and the conflict is to be stated, so it is filed for the parent and not as a defect to revert.
The diff documents the new kind of allowlist entry only in the guard's doc comment.

### [MEDIUM] A transport failure on a re-read is unchecked

**Location:** `internal/fabricengine/executor.go:156`, `internal/fabricengine/executor_test.go`

**Issue:** The only re-read error row uses `errors.New("disk error")`.
Returning every re-read error would pass every row, and an unreachable origin after an action list would then abort the verb instead of reaching the rule.

### [MEDIUM] A mixed joined read error is unchecked

**Location:** `internal/fabricengine/executor.go:74`

**Issue:** The one joined-error row holds two transport failures.
Recording the transport parts and swallowing the rest, or dropping the class condition, passes every row.

### [MEDIUM] The multi-store partial fill is unchecked

**Location:** `internal/fabricengine/snapshotread.go:37`

**Issue:** Card 15 states that a failing store does not stop the walk.
`TestSnapshotReader_ReadsAHub` has all-success reads and a single-store failure, and `TestRunRule` fakes the read.

### [MEDIUM] The records-store field is half checked

**Location:** `internal/fabricengine/snapshotread.go:129`

**Issue:** No test wants non-empty `Heads` or `Tags`, and none reads an absent records directory.

### [MEDIUM] The fabric test code barely shrinks

**Location:** the `_test.go` files of `internal/fabricengine`, `internal/fabriclayout`, `internal/fabriccore` and `internal/testkit/fabrickit`

**Issue:** Measured with `wc -l`: 58594 at the base, 58200 at HEAD (`fabricengine` 57286, `fabriclayout` 728, `fabriccore` 74, `fabrickit` 112), matching commit `f4fd75355`.
The later added entry's net shrink is met, and the plan estimated a smaller one.
The earlier added entry still asks review to raise a part that barely shrinks.

### [MEDIUM] Deleted tests without a Loosened-Check trailer

**Location:** `internal/fabricengine/destroy_test.go`, and the files of the other tests the fix names

**Issue:** The range carries `Loosened-Check:` trailers for some deleted and trimmed tests, but for none of the tests listed in the fix.
For six of them the holder the commit body names is in the same file as the deleted test, and two are cut as guarding nothing.
The rubric exempts only an assertion that a test in another file still holds.

### [LOW] Layout comments speak of engine internals

**Location:** `internal/fabriclayout/mutation.go:16`, `internal/fabriclayout/verbresults.go:295`

**Issue:** The comments moved unchanged and now name symbols a reader of the leaf package cannot see.

### [LOW] Smaller uncovered branches

**Location:** `internal/fabricengine/executor.go:117`, `internal/fabricengine/snapshotread.go:105`

**Issue:** Removing the duplicate-phase check, the nil-lock guard, the tag-object filter of the git path or the prunable skip fails no test.

## Rejected

- Anchors read fails whole on a records worktree with an unborn or unreadable `HEAD` (code-correctness F1): fabric's records worktrees are created with commits, and an unreadable worktree answering with an error is the informative result; no rule consumes the field yet.
- A re-read that fails as transport leaves the older value marked as read (code-correctness F2): the reader's doc states that what failed stays as it was, and the rule is handed the read failure.
- A held records store with a missing directory answers differently per field (code-correctness F3): card 15 specifies the `Present` answer for that one field, and the caller chooses which stores the reader holds.
- A tag of a tag may differ between the two ref backends (code-correctness F4): rests on a reading of the git manual that no run confirmed.
- Nil `sleep` default untested (part of code-tests F7): running it means waiting in real time or adds a zero-delay case that asserts nothing.
- `TeardownReport.Error` is an `error` field in the layout: card 7 moves the type by name, and it is the seam's report, not an error type or a callback.
