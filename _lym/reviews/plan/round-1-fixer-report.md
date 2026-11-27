---
dispositions:
  - id: F1
    disposition: fixed-differently
    reason: "The wants build from lymdirs.BoltDirName with literal segments, as proposed, but each goes through filepath.FromSlash, since the moved functions join with filepath.Join and a forward-slash want fails on Windows; the self-check's fix-proposal lens found it."
  - id: F2
    disposition: fixed-differently
    reason: "TestHubScratchAndLogsDir is deleted and TestHubPaths holds its joins and sibling rule as proposed, but TestHubLogsDir_MkdirAllIdempotentAgainstFabricCreatedDotLym is deleted too rather than kept: it checks os.MkdirAll's own contract and guards no fabric rule, and the parent's net-shrink ruling asks for such deletions. hubscratch_test.go is removed."
  - id: F3
    disposition: ruling
    reason: "Parent decision 'A chain part may leave production code that only a named later part first calls' carried out in the overview and cards 13, 14 and 15."
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
  - id: F10
    disposition: fixed
  - id: F11
    disposition: fixed
  - id: F12
    disposition: ruling
    reason: "Parent decision 'This part's fabric test code shrinks in net' carried out in the overview and cards 2, 3, 4, 13, 14 and 15."
  - id: F13
    disposition: fixed
  - id: F14
    disposition: fixed
---

# Round 1 fixer report: plan

## Changes

- **F1** (fixed differently, card 3): `TestHubPaths`' wants are built as `filepath.FromSlash("/h/" + lymdirs.BoltDirName + "/_lym/specs")`, with no literal `_bolt`, and the card says why.
  I did not make the proposed bare `"/h/" + lymdirs.BoltDirName + "/_lym/specs"`, because `filepath.Join` gives backslashes on Windows, where lym runs (`internal/reedengine/doc.go` names psmux there).
  `FromSlash` keeps the literal segments, while a `filepath.Join("/h", lymdirs.BoltDirName, "_lym", ...)` want would put an exact `_lym` geometry token in a join.
- **F2** (fixed differently, card 3): `TestHubScratchAndLogsDir` is under **Delete:**, and `TestHubPaths` holds its joins and the sibling rule as literal `HubScratchDir` and `StencilsDir` rows.
  `TestHubLogsDir_MkdirAllIdempotentAgainstFabricCreatedDotLym` is deleted too (see F12), and the emptied `hubscratch_test.go` is removed.
- **F4** (card 14): `runRule` takes `lock func(phase fabriccore.Phase) (release func(), err error)`.
  The Intent covers the lock call before a phase's first read, holding every release until return on every path in reverse order, the re-read under the held lock, a nil `lock` and a lock error.
  After the self-check, the lock is taken once per phase, so a phase asked for again reads under the lock it already holds.
  `TestRunRule` gains rows for the lock taken and released, a refusal after an under-lock phase, a lock error and a phase asked for twice.
- **F5** (cards 14, 15): `fieldReadError` with `Error` and `Unwrap` is under card 14's **Create:**.
  `runRule` reads it through `errors.As`, fills `ReadFailure`'s Phase, Field, Store, class and text, and returns a non-`fieldReadError` error as it stands; the text-matching sentence is gone.
  Card 15 uses the type, keeps the inner message, and stops a multi-store field at its first failing store.
  The transport row asserts Store and Class.
  `fieldReadError.Error` and `Unwrap` take allowlist entries, since they have results and no main reaches them.
- **F6** (card 12): the Store glyphs are `const StoreCode Store = fabriclayout.RepoCode` and its two siblings, which compiles since the layout constants are untyped (`internal/fabricengine/failedpush.go:53-60`).
  The Store bullet names the layout's values.
- **F7** (cards 12, 13): `BackoffDelays` is dropped from `Snapshot`, and the `Builder.BackoffDelays` Create line, its setter mention and the kit's prose mention are dropped too; card 12 says the delays land with part 2's first wait rule.
- **F8** (card 15): `ForgetOriginNamespaceForTest` is under **Uses:**, and the `ErrRecordsLayout` row unsets the fetch refspec and forgets the cached layout before the read.
- **F9** (card 1): `recordingTB` overrides `Helper` (a no-op), `Error`, `Errorf` and `Fatalf`.
  `AssertNoImports` returns right after its `Fatalf`, so the missing-directory row records exactly one fatal message.
- **F10** (cards 13, 14): card 13 edits `cmd/lym#testOnlyCodeAllowed` for `internal/fabriccore.Refusal.Error` and carries the proposed ImpactSummary; card 14 drops that entry.
- **F11** (cards 2, 10, 11): `TestScopedPathspec` moves in card 2, and `TestCommitResult_Committed` moves in card 10, which removes `committed_test.go` and excepts the test from the verbs-stay sentence.
  The affected-set rerun found one more moved pure name's test, `TestOrigin_JSONRoundTrip`, which card 11 now moves, removing `origin_test.go`.
- **F13** (cards 2, 3, 4): card 3 edits `internal/reedengine/doc.go` and rewrites the `_bolt` owner comment, card 4 edits `pattern/PATTERN-single-repo-hub.md`, and card 2's Intent names `structuraldirs_test.go`'s file header.
  After the self-check, card 3 also names `junctionnames_test.go`'s file header.
- **F14** (card 14): `livestate_doc_test.go` is under **Edit:** and names both meanings, and the doc.go executor section sets the core's snapshot apart from snapshot tags and the recorder's `Snapshot`.

`lym webster validate` reports one blocking finding, `plan-unapproved`, which is expected while the plan is under review; its other findings are informational `create-new-unit` lines.

## Self-check

- fix-consistency, LOW: the allowlist doc comment names the temporary kind of entry only from card 14, one card after card 13 adds the first one — deferred, see below.
- fix-consistency, LOW: `junctionnames_test.go`'s file header goes stale under card 3 — fixed: card 3 says the header names only the tests that stay.
- fix-claims, MEDIUM: card 3 said `TestConfigTemplate` pins the wired set, but it checks a copy of the formula — fixed: the bullet now says the empty template pathspec and `TestFilterHubReserved` hold the routes, and names `WiredNames`' disk load as what the deletion leaves unchecked.
- fix-proposal, MEDIUM: `lock` called on every phase request would take a non-reentrant lock twice — fixed: the lock is taken at a phase's first request, and the transport row asserts that a second request of the phase takes no second lock.
- fix-proposal, MEDIUM: forward-slash wants fail on Windows — fixed differently: the wants use `filepath.FromSlash` rather than the lens's proposed `filepath.Join` with a `"_lym"` argument, which would put an exact `_lym` geometry token in a join.
- fix-proposal, LOW: the same doc-comment timing as the first fix-consistency finding — merged with it, deferred.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

- Self-check finding on the doc-comment timing (fix-consistency and fix-proposal, LOW).
  The parent's decision "A chain part may leave production code that only a named later part first calls" says card 14 updates `testOnlyCodeAllowed`'s doc comment.
  F10 moved the first temporary entry to card 13, so the comment at `cmd/lym/testonlycode_linux_test.go:41-46` is incomplete for one card.
  Moving the comment edit to card 13 would depart from the recorded decision, so I left it in card 14. If the parent wants it moved, that takes a `lym loom decision replace` naming card 13.

## Rulings

- **F3**: the ruling admits production functions that only tests reach until a named later part adds their first caller.
  Each gets a `testOnlyCodeAllowed` entry whose reason names that part's slug, and that part removes the entry.
  Card 14 updates the doc comment, and cards 14 and 15 cite the decision instead of a self-declared departure.
  Done:
  - The overview's allowlist decision now cites the added entry and drops its "Departure" line.
  - The overview lists every such function: `runRule`, `outcomeClassOf`, `fieldReadError.Error`, `fieldReadError.Unwrap`, `snapshotReader.read`, `readLocalRefs` and `fabriccore.Refusal.Error`.
  - Each entry's reason names `fabric-core-push-origin`, the board slug of part 2, read with `lym board find fabric-core`.
  - Card 14 updates the doc comment to name the second, temporary kind of entry.
  - Cards 13, 14 and 15 cite the added entry.
  Nothing is left undone, and I sent the parent no flag.
- **F12**: the ruling asks for three things:
  - card 2 cuts `TestStructuralDirs`' literal-member rows;
  - each card's commit body names the test functions it added and deleted;
  - the part ends with less fabric test code, and the plan names the new code each added test alone guards.

  Done:
  - Card 2: `TestStructuralDirs` keeps only its no-aliasing row.
  - The overview's test decision now states the net-shrink rule. It requires each card's commit body to name added and deleted tests, and lists every added test with the new code it alone guards.
  - Card 15's commit body records the measured `_test.go` line totals at the part's base and after card 15. If the total grew, the implementer stops and asks the parent.
  - New deletions, each needing no successor or held by a named test:
    - card 3: `TestHubLogsDir_MkdirAllIdempotentAgainstFabricCreatedDotLym`, `TestHubReserved_BoardExcludedFromPathspecRoutes` and `TestOriginRecordPaths`; `TestIsReservedHubName` loses its three duplicate subtests, and `TestFilterHubReserved` loses its two member loops;
    - card 4: `TestWeftPathAccessors` loses its pairing tail.
  - Smaller new tests:
    - `TestBuilder` has two rows;
    - `TestRunRule`'s scenario rows are merged;
    - the reader's anchor row is folded into its first hub row.

  Nothing is left undone, and I sent the parent no flag.
  I did not measure the line totals myself, because the cards are not yet written as code, so the plan has the implementer measure them in card 15.
