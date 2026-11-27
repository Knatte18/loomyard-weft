---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    origin: lens:affected-set
    location: _lym/plan/03-layout-hub-paths.md:102
    summary: "TestHubPaths wants literal paths such as `/h/_bolt/_lym/specs`, and TestEnforcement_BoltDirNameDeclaredOnce fails every whole-token `_bolt` in any Go file, tests included, outside internal/lymdirs/dirs.go (internal/lymcwd/enforcement_test.go:526-575, Filter scankit.All), so card 3 fails its own tier0 gate"
    fix: "In card 3's TestHubPaths bullet, build each want from `lymdirs.BoltDirName` concatenated with literal segments, for example `\"/h/\" + lymdirs.BoltDirName + \"/_lym/specs\"`, as card 2's TestClassifyPaths row already does, and say why: the Bolt directory name may be spelled only at its declaration. No literal `_bolt` appears in any want."
  - id: F2
    severity: BLOCKING
    class: scope
    origin: lens:affected-set
    location: _lym/plan/03-layout-hub-paths.md:102
    summary: "TestHubPaths plans one row per moved path function, but TestHubScratchAndLogsDir (internal/fabricengine/hubscratch_test.go:30) already pins HubScratchDir, HubLogsDir, LogArchiveDir and AdoptScratchDir, and card 3 neither moves nor deletes it, so the same joins are tested twice, against PATTERN-test-speed's 'pins behavior no other test covers'"
    fix: "In card 3, add `internal/fabricengine_test#TestHubScratchAndLogsDir` under **Delete:**. The Tests-shrink bullet says TestHubPaths' rows hold its joins and its 'scratch directory is a sibling of the stencils `_lym` component' assertion, as a literal row. TestHubLogsDir_MkdirAllIdempotentAgainstFabricCreatedDotLym stays in hubscratch_test.go, since it tests a MkdirAll over a real directory. (Checked with validate-plan --overlay.)"
  - id: F3
    severity: BLOCKING
    class: decision
    origin: handler
    location: _lym/plan/14-executor-loop.md:45
    summary: "The overview's Shared Decision and cards 14 and 15 put production functions that only tests reach (runRule, outcomeClassOf, snapshotReader.read, readLocalRefs, fabriccore.Refusal.Error) in production files with testOnlyCodeAllowed entries. PATTERN-test-only-code admits an entry only for a reference or helper other packages' tests share, or one the analysis cannot see; no Decision, constraint or added entry of the decision record licenses this departure, and the guard's doc comment and the background's Rule both go false (cmd/lym/testonlycode_linux_test.go:41-46)"
    fix: "Put it to the parent with `lym creel post --parent --expect-reply`: D9 delivers the executor loop with no caller, and PATTERN-test-only-code forbids an unreached production function, so the constraint wins until a recorded decision settles it. Ask for an Added after Discussion entry, or a declared PATTERN-test-only-code change, that admits a chain part's functions which a named later part first calls, bounded to that part and removed when the caller lands. Then cards 14 and 15 cite that entry in place of the self-declared departure, and card 14 updates testOnlyCodeAllowed's doc comment to name that second kind of entry."
  - id: F4
    severity: BLOCKING
    class: design
    origin: handler
    location: _lym/plan/14-executor-loop.md:24
    summary: "runRule always starts from an empty snapshot and has no seam to take the verb's lock between phases, so D2's under-lock phase, read 'once the verb holds the lock it takes today', and D3's re-read 'under the lock the verb holds at that step today' cannot be expressed. Every later part that drives the loop would first have to change its signature."
    fix: "Replace card 14's runRule glyph with `func runRule(rule fabriccore.Rule, start fabriccore.Answer, read snapshotRead, lock func(phase fabriccore.Phase) (release func(), err error), runners map[fabriccore.ActionKind]actionRunner, sleep func(time.Duration)) (fabriccore.Snapshot, error)`. The Intent says runRule calls `lock` with each phase before its first read; holds every release until it returns, on every return path, releasing in reverse order; and runs a re-read under the latest phase's held lock. A nil `lock` takes none, and a lock error returns like a raw read error. TestRunRule gains rows: an under-lock phase request takes the lock before its read and releases it at return, a refusal after it releases too, and a lock error is returned with no read. (Checked with validate-plan --overlay.)"
  - id: F5
    severity: BLOCKING
    class: consistency
    origin: handler
    location: _lym/plan/14-executor-loop.md:35
    summary: "Cards 14 and 15 leave the read-failure contract open. Card 15 only wraps each read error with %w 'naming the field and the store' in its text. Card 14 must recover a ReadFailure's Field from that text ('the first of the request's fields the error names') and never sets the Store or Class that card 12's ReadFailure holds. Two implementers would produce disagreeing code, and nothing says whether a multi-store field stops at its first failing store."
    fix: "In card 14, add `plan:internal/fabricengine#fieldReadError` -> `type fieldReadError struct`, `plan:internal/fabricengine#fieldReadError.Error` -> `func (e *fieldReadError) Error() string` and `plan:internal/fabricengine#fieldReadError.Unwrap` -> `func (e *fieldReadError) Unwrap() error` under **Create:**. The type holds Field, Store and Err. runRule takes them through errors.As, setting the ReadFailure's Phase from the request, its Field and Store from the error, its Class from outcomeClassOf(Err) and its text from Err, and drops the text-matching sentence. A read error with no fieldReadError is returned as it stands. TestRunRule's transport row asserts Store and Class. In card 15, add `plan:internal/fabricengine#fieldReadError` under **Uses:**. Every field read returns its error as a *fieldReadError naming the field and the store, keeping the inner message so the ErrRecordsLayout row still matches, and a field read over several stores stops at its first failing store. (Checked with validate-plan --overlay.)"
  - id: F6
    severity: BLOCKING
    class: consistency
    origin: handler
    location: _lym/plan/12-core-package.md:13
    summary: "fabriccore.StoreCode, StoreRecords and StoreBolt declare the literals code, records and bolt a second time; card 8 already declares them as fabriclayout.RepoCode, RepoRecords and RepoBolt for the same repository sides. That breaks PATTERN-code-conventions' 'one named constant, declared once', the rule card 9 itself cites to retire the engine's twins."
    fix: "In card 12, change the three Create glyphs to `const StoreCode Store = fabriclayout.RepoCode`, `const StoreRecords Store = fabriclayout.RepoRecords` and `const StoreBolt Store = fabriclayout.RepoBolt`. The Store bullet says each store's value is the layout's repository-side name. (Checked with validate-plan --overlay.)"
  - id: F7
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/plan/12-core-package.md:81
    summary: "Snapshot.BackoffDelays has no supplier. No Field names it, card 15's reader fills only the six fields, and runRule starts from an empty snapshot, so nothing outside a test sets it. That departs from the overview's own decision that a field no rule of this part reads lands with the part whose rule first reads it."
    fix: "In card 12, drop `BackoffDelays []time.Duration` from the Snapshot bullet. In card 13, drop the `plan:internal/testkit/fabrickit#Builder.BackoffDelays` Create line, its mention in the setter bullet and its TestBuilder row. Both land with part 2's first wait rule, as the overview's 'snapshot fields and the seam tests' second half land with their first rule' decision says. (Checked with validate-plan --overlay.)"
  - id: F8
    severity: MEDIUM
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/15-snapshot-reader.md:57
    summary: "The ErrRecordsLayout row cannot reach its state as written. originNamespaceAt caches each store's layout for the process (internal/fabricengine/recordslayout.go:67-70), and NewOneRepoHub's in-process clone has already cached the good answer, so breaking the fetch refspec still reads as the namespaced layout. The existing test of this state forgets the cache first (internal/fabricengine/onerepository_lifecycle_integration_test.go:49-50)."
    fix: "In card 15, add `internal/fabricengine#ForgetOriginNamespaceForTest` under **Uses:**. The fourth TestSnapshotReader_ReadsAHub row says it unsets the records store's `remote.origin.fetch` and then calls `fabricengine.ForgetOriginNamespaceForTest(h.PrimeRecords())` before the read. (Checked with validate-plan --overlay.)"
  - id: F9
    severity: MEDIUM
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/01-scankit-banned-imports.md:32
    summary: "recordingTB embeds a nil testing.TB and records only Errorf and Fatalf. AssertNoImports' t.Helper() call and RequireFloor's t.Helper() and t.Error calls (internal/testkit/scankit/scankit.go:281-286) panic through the nil embed, and a recorded Fatalf does not stop the call, so the missing-directory row records a second message."
    fix: "In card 1, say recordingTB overrides Helper (a no-op), Error, Errorf and Fatalf, recording each message and the failed state. AssertNoImports returns right after its Fatalf on a scan error, so the missing-directory row records exactly one fatal message and no floor message."
  - id: F10
    severity: MEDIUM
    class: design
    origin: lens:repo-invariants
    location: _lym/plan/13-fabrickit-snapshot-builder.md:22
    summary: "From card 13 on, internal/testkit/fabrickit, a test-support package, imports internal/fabriccore, which brings the core into the tier1 test-only-code analysis. No root reaches fabriccore.Refusal.Error, and its testOnlyCodeAllowed entry arrives only in card 14, so the guard fails at a batch boundary between cards 13 and 14 (pattern/PATTERN-test-only-code.md bounds, cmd/lym/testonlycode_linux_test.go:81-94)."
    fix: "Subject to F3's decision: in card 13, add `**Edit:**` with `cmd/lym#testOnlyCodeAllowed`, admitting `internal/fabriccore.Refusal.Error` with its reason. Add `**ImpactSummary:** One allowlist entry in the test-only-code guard, for fabriccore.Refusal.Error, which the kit's import first brings into the analysis; no production behavior changes.` Card 14 drops that entry from its own list. (Checked with validate-plan --overlay.)"
  - id: F11
    severity: MEDIUM
    class: design
    origin: lens:affected-set
    location: _lym/plan/02-layout-package-classify.md:16
    summary: "The overview says a test of a moved pure name moves to the layout at tier0, but the cards' sweep missed some. TestScopedPathspec (internal/fabricengine/fabric_test.go:159) and TestCommitResult_Committed (internal/fabricengine/committed_test.go:16) test ScopedPathspec and CommitResult.Committed, which cards 2 and 10 move, yet no card moves them, so they stay in the engine's test package and run on every engine change."
    fix: "In card 2, add `internal/fabricengine_test#TestScopedPathspec` under **Move:** and say it moves to the layout's tier0 tests unchanged. In card 10, add `internal/fabricengine_test#TestCommitResult_Committed` under **Move:**, with the emptied committed_test.go removed. Card 10's 'The verbs' tests stay' sentence then excepts that pure-method test. (Checked with validate-plan --overlay.)"
  - id: F12
    severity: MEDIUM
    class: decision
    origin: handler
    location: _lym/plan/02-layout-package-classify.md:80
    summary: "Under the added entry 'The restructure cuts fabric's test code', review raises a part whose test code grows. This part deletes eight small tests and adds about fifteen test functions, a tier1 seam test on real hubs among them. One of the new ones, TestStructuralDirs, pins members that TestIsReservedHubName's `_lym` row (internal/fabricengine/junctionnames_test.go:202) and TestStructuralNameSets' `.lym` checks (internal/fabricengine/structuraldirs_test.go:37-53) already pin."
    fix: "In card 2, cut TestStructuralDirs' literal-member rows, which those two tests hold, and keep only its row that appending to a returned slice leaves the next call's answer unchanged. With F2 and F11 applied, each card's commit body states the test functions it added and deleted. Where the part's fabric test code still grows, the plan names the new code each added test alone guards, so the parent can judge it against the added entry."
  - id: F13
    severity: LOW
    class: design
    origin: lens:repo-invariants
    location: _lym/plan/03-layout-hub-paths.md:60
    summary: "Prose that names a moved name's home is missed when it does not spell `fabricengine.<Name>`. Missed sites: internal/reedengine/doc.go:20 'The one fabricengine call, fabricengine.HubScratchDir' (card 3); the `_bolt` row comment of geometryTokenOwners, 'fabricengine and runproc take the name from it' (internal/lymcwd/enforcement_test.go:229, card 3); pattern/PATTERN-single-repo-hub.md:11 'spelled once, by fabric's RecordsNamespaceRoot' (card 4); and structuraldirs_test.go's file header naming structuralCommittedDirs (card 2)."
    fix: "In card 3, add `internal/reedengine/doc.go` under **Edit:**, saying the one call is `fabriclayout.HubScratchDir` and reedengine imports no fabricengine. The Intent rewrites the `_bolt` comment of TestEnforcement_GeometryLiterals, which card 3 already edits, to say the layout package takes the name. In card 4, add `pattern/PATTERN-single-repo-hub.md` under **Edit:**, naming fabriclayout's constants. In card 2, the Intent names the file header of internal/fabricengine/structuraldirs_test.go among the comments it rewrites; redundant-file-target bars listing that file beside the two members card 2 already lists. (Checked with validate-plan --overlay.)"
  - id: F14
    severity: LOW
    class: consistency
    origin: lens:repo-invariants
    location: _lym/plan/14-executor-loop.md:67
    summary: "internal/fabricengine/livestate_doc_test.go:22 says 'In fabric's own vocabulary, Snapshot names the `Snapshot: <tag>` trailer', and the engine's doc.go already uses snapshot tags and Mutations.Snapshot. Card 14's doc section makes 'snapshot' the core's input (D11) without telling the meanings apart."
    fix: "In card 14, add `internal/fabricengine/livestate_doc_test.go` under **Edit:**, whose sentence names both meanings: the `Snapshot: <tag>` trailer, and the core's input snapshot. The doc.go executor section says its 'snapshot' is the core's input, distinct from snapshot tags and the recorder's Snapshot. (Checked with validate-plan --overlay.)"
---

### [BLOCKING] Card 3's literal `_bolt` wants fail an existing tier0 guard
**Location:** `_lym/plan/03-layout-hub-paths.md:102`
**Issue:** `TestEnforcement_BoltDirNameDeclaredOnce` (`internal/lymcwd/enforcement_test.go:526-575`) walks every Go file with `scankit.All` and fails any whole-token `_bolt` outside `internal/lymdirs/dirs.go`.
`/h/_bolt/_lym/specs` matches its pattern, so card 3 cannot pass its own card gate.
Card 2 knew the rule and spelled its row through `lymdirs.BoltDirName`, while card 3 did not.

### [BLOCKING] TestHubPaths duplicates TestHubScratchAndLogsDir
**Location:** `_lym/plan/03-layout-hub-paths.md:102`
**Issue:** `internal/fabricengine/hubscratch_test.go:30` pins `HubScratchDir`, `HubLogsDir`, `LogArchiveDir` and `AdoptScratchDir`, and its comment explains why.
Card 3's new table takes one row per moved path function, and it deletes four other one-join tests but never names this one.
The result is the same rule tested twice, once in each package, against `PATTERN-test-speed` and the added entry on cutting fabric's tests.

### [BLOCKING] The dead-code allowlist departure has no licence
**Location:** `_lym/plan/14-executor-loop.md:45`
**Issue:** `PATTERN-test-only-code` is not among the task's declared PATTERN changes, and its Rule admits an unreached production function only as a helper other packages' tests share, or one the analysis cannot see.
The overview names the departure itself, and no Decision, constraint or Added after Discussion entry records it.
D9, which delivers the loop with no caller, and the PATTERN entry conflict here, and under this round's rules the constraint wins until the parent records a decision.
The repo-invariants lens's note that the guard's doc comment and the background go false is folded in here.

### [BLOCKING] The executor loop cannot hold a verb's lock between phases
**Location:** `_lym/plan/14-executor-loop.md:24`
**Issue:** D2 reads the under-lock phase once the verb holds today's lock, and D3 re-reads stale fields under the lock held at that step.
`runRule` starts from an empty snapshot and calls only `read`, `runners` and `sleep`, so a verb cannot take its lock after the pre-lock phase and still carry that phase's snapshot.
The card's stated role is "the executor loop every later part drives", so the shape has to carry the lock seam now.

### [BLOCKING] Cards 14 and 15 leave the read-failure contract unspecified
**Location:** `_lym/plan/14-executor-loop.md:35`
**Issue:** Card 15's errors carry the field and the store only in their message text.
Card 14 attributes a failure to "the first of the request's fields the error names", which is text matching, and it never fills `ReadFailure.Store` or its class.
A typed wrapper defined in the earlier card settles the order and the contract.

### [BLOCKING] fabriccore redeclares the repository-side literals
**Location:** `_lym/plan/12-core-package.md:13`
**Issue:** `StoreCode = "code"`, `StoreRecords = "records"` and `StoreBolt = "bolt"` sit beside card 8's `fabriclayout.RepoCode`, `RepoRecords` and `RepoBolt`, which hold the same values for the same sides.
`PATTERN-code-conventions` wants one named constant, and card 9 retires the engine's twins for exactly this reason.

### [MEDIUM] Snapshot.BackoffDelays has no supplier
**Location:** `_lym/plan/12-core-package.md:81`
**Issue:** No `Field` names it, the reader leaves it unfilled, and `runRule` starts empty, so only the kit's setter ever sets it.
The overview's own snapshot-fields decision defers such fields to the part whose rule first reads them.

### [MEDIUM] The ErrRecordsLayout row reads a cached layout
**Location:** `_lym/plan/15-snapshot-reader.md:57`
**Issue:** `originNamespaceAt` keeps a store's answer for the process, and `NewOneRepoHub`'s in-process clone has already cached the good answer.
The row needs `ForgetOriginNamespaceForTest` after it breaks the refspec, as `onerepository_lifecycle_integration_test.go:49-50` does.

### [MEDIUM] recordingTB panics through its nil embedded TB
**Location:** `_lym/plan/01-scankit-banned-imports.md:32`
**Issue:** `RequireFloor` calls `t.Helper()` and `t.Error` (`internal/testkit/scankit/scankit.go:281-286`), and neither is recorded by the planned type.
A recorded `Fatalf` also lets the call run on into the floor check.

### [MEDIUM] Refusal.Error's allowlist entry lands one card late
**Location:** `_lym/plan/13-fabrickit-snapshot-builder.md:22`
**Issue:** The kit is a test-support package, and its import brings fabriccore into the tier1 analysis at card 13.
The entry arrives at card 14, and at card 12 it would be a stale entry, so card 13 is its only home.

### [MEDIUM] Engine tests of moved pure names stay behind
**Location:** `_lym/plan/02-layout-package-classify.md:16`
**Issue:** `TestScopedPathspec` and `TestCommitResult_Committed` test functions cards 2 and 10 move, yet stay in the engine's external test package.
This breaks the overview's "Decision: the foundation's tests shrink", which moves such tests to the layout at tier0.

### [MEDIUM] The part's test code grows against the added entry
**Location:** `_lym/plan/02-layout-package-classify.md:80`
**Issue:** The added entry asks review to raise a part whose fabric test code grows or barely shrinks.
This part adds far more test functions than it deletes, and the cards and overview give no count.
`TestStructuralDirs`' member rows are one concrete duplicate.
Whether the remaining growth is accepted as new code's own tests is the author's call, made visible to the parent.

### [LOW] Prose naming a moved name's home is missed
**Location:** `_lym/plan/03-layout-hub-paths.md:60`
**Issue:** The cards' comment rule rewrites `fabricengine.<Name>` spellings only.
Prose that names the home in other words goes false after the move: reedengine's doc, a geometry-owner comment, a PATTERN background and a test-file header.

### [LOW] Two meanings of "snapshot" in the engine's docs
**Location:** `_lym/plan/14-executor-loop.md:67`
**Issue:** The live-state harness doc reserves "Snapshot" for the trailer, while card 14 adds the core's input snapshot to the engine's doc without telling the two apart.

## Rejected

- Focus entry on cards 6 and 9: package-glyph Edit targets are no defect. The member-glyph bullet fires only for file-path targets, and `redundant-package-target` refuses a member glyph beside the package glyph each card needs for its package-wide retarget, so no re-sign arrow or Delete member can be added.
- guard-reach finding 3, no ref-storage fallback for `FieldAnchors` and `FieldRecordsStore`: D2 asks for the fallback for the worktree list and refs alone, and today's reads behave the same, so the card meets the record.
- affected-set F5's `junctionnames_test.go:253-254` and `structuraldirs_test.go:25,28`: the first lies in `TestIsReservedHubName`, whose comment card 3 rewrites as it moves it; the second is `TestStructuralNameSets`' comment, which card 2 rewrites. Only the file header remains, and it is kept in F13.
- affected-set F3's seed and sync subject rows of `TestSeedCommitSignature`: they pin the subject the seed commit carries together with the engine's own `binaryLabel`, which stays, so the rows guard a joint engine output.
- repo-invariants F3's burlerengine, fabriccli and pairteardown doc lines: each spells `fabricengine.<Name>` and is covered by the cards' comment rule.
- Focus entry on cards 2–11 and the closed import set: every moved declaration's fields, method bodies and referenced constants compile inside `internal/fabriclayout` at their card. The one engine dependency is the BranchOutcome way-forward methods, which card 7 keeps as engine functions. Not a defect.
- Focus entry on cards 1, 2 and 12: the two leaf tests fail for `gitexec`, `gitrepo`, `dotgit` and `lymcwd` through the allowlist, and for `os/exec` through `os/...`. Not a defect.

## Focus departures

- "Cards 2-11 against D1, D7 and the overview's 'what the layout takes' decision: … and does a file that should drop its engine import keep it for such a name?" I compared every exported engine name that files outside fabric use against the Move lists. No D1-kind name is left behind; the remaining plain-looking types hold a `*lymcwd.Location`, a function field or a seam. I did not check each of the roughly 470 importing files to see whether it drops its `internal/fabricengine` import, since the gate's build and goimports settle that per file and a manual sweep would only restate them.
