---
verdict: BLOCKING
findings:
  - id: F19
    carries: F18
    severity: BLOCKING
    class: design
    location: "_lym/plan/12-board-test-moves.md:107"
    summary: "Card 12 still restates in boardengine_test support that stays in use in fabricengine_test: recordsFixture's primeWeft, originCommit and endEntryOnOrigin, with newReplayHub and addPushedPair repeating newRecordsFixture and newPairFixtureAt. PATTERN-test-isolation says shared support is never duplicated per package, and a kit's gitkit ban does not force a second copy, since gitkit itself can export the git plumbing"
    fix: "In card 12, remove the Create lines of `plan:internal/boardengine_test#newReplayHub`, `#pushPrimeRecords`, `#commitOnOrigin`, `#endEntryOnOrigin` and `#addPushedPair`. Add the Create line `plan:internal/gitkit#CommitOnBare` -> `func CommitOnBare(tb testing.TB, bare, branch, rel, content, message string) string` (clone bare at branch into a temp dir, commit rel, push, return the sha). Add the Extends line `internal/gitkit/doc.go`, and the Edit lines `internal/fabricengine_test#recordsFixture.originCommit` and `internal/fabricengine_test#originCommitFile`, both of which call the new export, so one copy stands. Replace the restating sentences and the newReplayHub bullet: moved rows call boardkit.InstallReplay and hubforge.CopyHub; they add a pair with hubforge.AddPair, which already pushes both sides (AddPairWith in internal/hubforge/hub.go), then lymcwd.ResolveWorktree; they publish the prime's records with one gitkit.Git `push --set-upstream`; they write origin's board with gitkit.CommitOnBare over boardkit.BoardFile. originCommit sits in replay_integration_test.go, another concern's file, so name that edit as a departure from D9's own-files criterion. Unverified: that no moved row needs heddle's replayer; a row that changes heddle.json registers it in its own body. `lym loom validate-plan --overlay` accepts these target changes applied together with F20's."
  - id: F20
    severity: BLOCKING
    class: design
    location: "_lym/plan/12-board-test-moves.md:109"
    summary: "The new kit internal/testkit/pushkit holds assertions on the code under test (RequireOneFailedPush, RequireNoFailedPush, RequireEntry). PATTERN-test-isolation says a kit asserts nothing beyond t.Fatalf on its own setup, and pushkit is none of the exceptions it lists"
    fix: "In card 12, remove the four `plan:internal/testkit/pushkit#` Create lines. Remove from Delete `requireOneFailedPush`, `requireNoFailedPush`, `failedPushesOf`, `countingBackoff` and `requireEntry`. Remove from Edit the lines that only retarget a call: `requireRewoundHoldBack`, `requireNoRecordOfUnrecordedPush`, `TestRefusedPushStands`, `TestPushPrimeCodeTip`, `TestBolt_PushRecorded_DropsSeedCommitsOnRebaseConflict`, `divergedPair.requireNothingPushed` and `TestRejectedRecordsPush_ReadsTheTakeover`. Delete the pushkit bullet with its D9 departure for the three other concerns' files. State that fabric's helpers stay as they are, and that each moved row asserts inline over fabricengine.ListFailedPushes and the report's Entries and counts backoffs with an inline closure, adding no helper to boardengine_test. Drop pushkit from the `teardownRecorder` bullet and the kit from the ImpactSummary. `lym loom validate-plan --overlay` accepts these target changes applied together with F19's."
  - id: F21
    severity: BLOCKING
    class: design
    location: "_lym/plan/12-board-test-moves.md:135"
    summary: "Card 12 carries into boardengine_test a teardownRecorder that scripts pairteardown's reports, and scripted OwnerOf answers, and asserts the fabricated teardown entries as the reconcile's printed mutations. D12 bars feeding fabric another engine's made-up output in a receiving package, and PATTERN-test-stand-ins bars a hand-written function in a seam another module fills"
    fix: "In card 12, keep the Create line of `teardownRecorder` but define it as a recording wrapper: it records each EndedTeardown and passes it to the real seam, and it has no reply field. The moved rows build that seam with `pairteardown.New(h.Location)` and `OriginSeams(pairteardown.HubSeamOptions(h.Location))`, both exported, and leave LivePair nil. Drop the assertions on scripted Tags, Kept and mutation entries; card 10's TestEndedTeardowns holds the copying of a report, and the printed mutations asserted are those the real teardown records. The owned_elsewhere rows take OwnerOf from the same real seams, or from `pairteardown.OwnerReader` with a fixed identity, over owners recorded in the board's own entries; the OwnerOf-error row provokes the real reader's error, such as by an owner read over board bytes that do not decode. The empty-subject row goes where card 9's TestClassifyEntries holds its rule. State the same in the two test descriptions. No target changes. Unverified: that the real teardown of these pairs runs in tier1 with no tmux server, what machinename.ReadIdentity answers in the test binary, and which board fields the real owner reader needs."
prior:
  - id: F15
    status: resolved
    answer: "Card 7 lists both tests and says how each sets the seam; card 3's account matches what its search returns."
  - id: F16
    status: resolved
    answer: "Settled by the added entry `A protected refusal of the leased reset keeps today's other record`, which card 7 carries."
  - id: F17
    status: resolved
  - id: F18
    status: open
    answer: "The notifier fake and the fixture types are gone. The git-using helpers are still restated, which F19 carries."
---

# Plan review, round 3

F15 to F17 are resolved and F18 is partly open.
Three findings block, all in card 12.
They come from reading the plan, the code and the PATTERN backgrounds;
no test was run, since the planned code does not exist yet.
The one thing run was the plan gate's overlay, over the target changes of F19 and F20 together, and it passed.

Focus entries:

- Card 7's refusal: not a defect.
  The in-package tests that reach `coalescePushSides` are the two card 7 gives the seam to.
  `TestDetachedPush_CoveredChildExitsBeforeItsFlags` returns before it,
  the role-guard push rows stop at worktree resolution, and the outside hits run over `hubforge` hubs.
- Card 7's protected refusals: not a defect.
  The first push and the replay pushes wrap `ErrPushRejected` and the leased reset does not,
  and each path's class, text and judgment read match the record with its added entries.
- Card 12's helpers: `writeRunStatus` now writes shed's own types, and no notifier fake or fixture type remains.
  The restated support is F19 and the scripted seams are F21.
- Card 12's exports: each named export exists or has a Create line, both consumers import without a cycle,
  and every fabric caller of a moved helper is in the card's targets.
  The new kit's assertions are F20.

### [BLOCKING] Git-using support is still duplicated

**Location:** `_lym/plan/12-board-test-moves.md:107`

**Issue:** The card says the helpers fabric keeps are restated in `boardengine_test`.
`PATTERN-test-isolation` forbids a copy per package, and `PATTERN-leaf-packages` names `gitkit` as the home of test git plumbing.
The kit import ban rules out `internal/testkit` for these helpers, not a single shared copy.

### [BLOCKING] A kit that asserts

**Location:** `_lym/plan/12-board-test-moves.md:109`

**Issue:** `pushkit` is described as holding the push and origin-reconcile assertions.
The Enforcement section of `PATTERN-test-isolation` limits a kit to `t.Fatalf` on its own setup and lists its exceptions.

### [BLOCKING] Scripted teardown and owner seams in the receiving package

**Location:** `_lym/plan/12-board-test-moves.md:135`

**Issue:** Production fills `OriginSeams.Teardown` and `OwnerOf` from `internal/pairteardown`.
Today's `teardownRecorder` answers hand-written tags, kept lines and mutation entries,
and card 12 recreates it in `boardengine_test` with an exact `mutations` assertion that includes those entries.

## Rejected

Seen and not raised: each is below BLOCKING or sits off the changed hunks, which this incremental round does not take.

- Cards 11 and 12 account for neither the anchor-marker row nor the records-replay-conflict row of `TestReconcileOrigin_Moves`.
  Both assert an entry, which needs the run record `movesTemplate` stops writing.
- Card 12 keeps fabric's `writeStatus`, a hand-written shed record, for one `TestBoltReplayers` row (D12).
- Card 12 keeps `plantRunTransition`'s shed record under `TestOriginCodeTip` and the owner-guard rows,
  neither of which reads it after card 4 (D12).
- `RecordsLandingVerdict` returns only a `PushFailureClass`,
  which cannot tell a landed side, an ended entry and an unclassified lookup failure apart.
- No card plans a seam test that asserts the snapshot of the `EndedTeardowns` flow (D6).
- `originStatusBytesAtTip` returns no path, so a corrupt record's parse error can no longer name it.
- Card 7 says `refuseUnseededBoard` has four ways forward, and the code has more.
- Card 7 lists `holdBackRewound`, which writes the rewound record itself, under no target.
- Card 7 does not say that an unreadable branch name returns the raw push result, with no retry or record.
- Card 8 edits `readLocalBranch` to read by the caller's question and gives no re-sign arrow.
- Cards 9 and 10 leave `branchAnchorStopWayForward`, `branchDirtyWayForward` and the engine's `busyWayForward`
  with no caller and no Delete line.
- Card 9 counts fifteen precedence rows where the test has fourteen.
- Card 3 creates `hubforge.InstallRequiredSeams`, which no card calls,
  and widens a tier0 test with no time estimate.
- The added entry on import sets says `boardkit` gets only D7's retarget, while card 12 adds `boardkit.BoardFile`.
  The one-kit rule of `PATTERN-test-isolation` wins, so no finding asks for its removal.
