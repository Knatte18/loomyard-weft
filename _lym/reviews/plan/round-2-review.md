---
verdict: BLOCKING
findings:
  - id: F15
    severity: BLOCKING
    class: design
    location: _lym/plan/00-overview.md:69
    summary: "The plan does not show that the part's fabric test code shrinks, which the added entry 'This part's fabric test code shrinks in net' requires. It leaves that to be found at card 15, where the implementer must halt and ask the parent if the total grew (00-overview.md:71-72). The named deletions of cards 2, 3, 4 and 15 come to under 600 lines of today's test files, and the planned additions look clearly larger."
    fix: "Settle the count now, not at card 15. Estimate the added _test.go lines of internal/fabricengine, internal/fabriclayout, internal/fabriccore and internal/testkit/fabrickit, new file headers and export_test.go accessors included. Name enough further internal/fabricengine test deletions, with a margin, on the cards that carry them, each with the test that holds its assertion or why it guards no rule of its own, and check any added Delete target with lym loom validate-plan --overlay. If no such deletions exist, put that to the parent and ask for an amended added entry. In 00-overview.md cut the clause that starts 'when the total grew, the implementer stops before that commit and asks the parent'; card 15's commit body keeps reporting the measured totals."
  - id: F16
    severity: MEDIUM
    class: consistency
    location: _lym/plan/15-snapshot-reader.md:49
    carries: F5
    summary: "The read-failure contract is still open in three places. Card 15 stops a multi-store field at its first failing store but gives no store order, and the stores are a Go map, so the recorded Store and the partly filled snapshot vary between runs. It names no Store for FieldPendingTeardowns, which reads the hub alone. Card 14 (14-executor-loop.md:34-48) does not say whether a phase whose read failed enters Snapshot.Phases, or which Phase a failed re-read records."
    fix: "In card 15, say read walks the stores in the fixed order StoreCode, StoreRecords, StoreBolt, skipping those the reader does not hold. Say which Store each other field's error carries: FieldWorktrees StoreCode, FieldAnchors and FieldRecordsStore StoreRecords, FieldPendingTeardowns the zero Store. In card 14, say a phase enters Snapshot.Phases only after its read succeeds, and a ReadFailure from a re-read records the phase the re-read ran under. TestRunRule's transport row then asserts Phases accordingly."
prior:
  - id: F1
    status: resolved
  - id: F2
    status: resolved
  - id: F3
    status: resolved
    answer: "Settled by the added entry 'A chain part may leave production code that only a named later part first calls'. The overview and cards 13, 14 and 15 carry it."
  - id: F4
    status: resolved
  - id: F5
    status: open
    answer: "The proposed fix is in cards 14 and 15, but two implementers can still disagree; F16 carries the rest."
  - id: F6
    status: resolved
  - id: F7
    status: resolved
  - id: F8
    status: resolved
  - id: F9
    status: resolved
  - id: F10
    status: resolved
  - id: F11
    status: resolved
  - id: F12
    status: resolved
    answer: "Settled by the added entry 'This part's fabric test code shrinks in net'. Whether the plan meets that entry's net requirement is F15."
  - id: F13
    status: resolved
  - id: F14
    status: resolved
---

### [BLOCKING] The net shrink is left to a halt at card 15

**Location:** `_lym/plan/00-overview.md:69-72`

**Issue:** The rubric makes a finding of a card that leaves a needed fact to be found while it runs, or that halts on discovering it.
The shared decision does both: it asserts that the deletions of cards 2, 3, 4 and 15 pay for the new tests, then has the implementer stop and ask if they did not.

The deletions, read from today's files, are:

- `hubscratch_test.go`, `specsdir_test.go`, `stencilsdir_test.go` and `hubreservedroutes_test.go` whole;
- the partition test and its helper in `classify_test.go`;
- `TestBoardDir`, `TestHubPath`, `TestOriginRecordPaths` and `TestList_RefStorage`;
- a few smaller functions, subtests and loops.

Together they are under 600 lines.
The additions include `TestRunRule`'s table, two tier1 reader tests, `TestBuilder`, `TestOutcomeClassOf`, `TestHubPaths`, the layout and core tests and their new files.
I estimate them at 800 lines or more.
That is an estimate, since none of them is written, but nothing in the plan shows the opposite.
`TestHubPaths` adds to the cost: its one row per path function covers several functions that none of the deleted tests held.

### [MEDIUM] The read-failure contract still lets two implementers differ

**Location:** `_lym/plan/15-snapshot-reader.md:48-49`, `_lym/plan/14-executor-loop.md:34-48`

**Issue:** "First failing store" has no meaning without an order.
With an unreachable origin every store fails, so a map walk records a random one, and a rule that reads `ReadFailure.Store` decides on it.

## Constraint conflict

`PATTERN-test-only-code` says every production function is reachable from a `main`.
Cards 13 to 15 add functions that only tests reach.
The added entry "A chain part may leave production code that only a named later part first calls" settles this for the run, so I raise no finding.
`pattern/PATTERN-test-only-code.md` still does not name this temporary kind of allowlist entry, no card updates it, and no PATTERN change is declared for it.

## Seen but not admissible as findings

This round is incremental, so a new finding must be `BLOCKING` and sit on a changed hunk.
The first item below would stop the run and I recommend that the parent or the fixer takes it up.

- **Would fail card 15, on unchanged lines (`15-snapshot-reader.md:65-72`).**
  `TestSnapshotReader_RefStorage` reads `FieldLocalRefs` from a subdirectory of a linked worktree.
  For a `files` store, `readLocalRefs` reads through `gitrepo`, whose open refuses a path that is not itself a repository (`internal/gitrepo/gogit.go:58-101`, `internal/gitrepo/doc.go:22-25`).
  The `files` row therefore errors as planned.
  Reading refs from the worktree root, and keeping the subdirectory read for `FieldWorktrees` alone, avoids it.
  This comes from reading the code; I ran no test.
- `TestStructuralDirs` (card 2) cannot fail: appending to a one-element slice with no spare capacity always allocates, so a shared package slice passes it.
  Overwriting element 0 as well would catch that.
- Card 3 asks `TestIsReservedHubName` rows to hold `HubReservedNames`' exact members and its lack of `.lym`.
  A predicate row cannot: `IsReservedHubName(".lym", nil)` is true.
  It needs an assertion on the slice.
- `TestBuilder`'s second row (card 13) catches shared maps only, since a later append never changes the elements an earlier snapshot sees.
- Card 4 leaves the seed and sync subject rows of `TestSeedCommitSignature` in the engine, though they test two moved pure names.
- Card 6 removes `Snapshot` and `Extend` from the record value and names no replacement where code calls them on one:
  `internal/pairteardown/teardown.go:582-583`, `internal/fabricengine/originreconcileended.go:125` and `internal/fabricengine/originreconcile.go:815`.
  `MutationsOf` over both entry lists would serve.

## Focus verdicts

- Cards 14 and 15, test-only code: the added entry licenses each allowlist entry, and no declared PATTERN change does.
  By reading the code, the guard holds from card 13 on.
- `runRule` against D2 and D3: the stated signature expresses the phases, the lock and the re-read, and releases on every return path.
  The one gap is in F16.
- Read-failure contract: F16.
- `fabriccore` types: each `Snapshot` field has a supplier in this part, and I found no constant repeating a layout name.
- Tests of moved names: no planned literal spells the Bolt directory name, and the gaps are listed above.
- Rows in cards 1 and 15: card 1's rows and the `ErrRecordsLayout` row can reach their states; the ref-storage row cannot, as listed above.
