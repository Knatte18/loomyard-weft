---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    location: _lyx/plan/02-audit-ledger.md
    summary: Batch-level audit warnings are erased when BeginBatch or recoverSpawn replace the batch record, while the ledger keeps the identity dispositioned, so the warning is lost for good
  - id: F2
    severity: MEDIUM
    location: _lyx/plan/08-record-batch-audit-dispositions.md
    summary: Card 8's no-duplicate test calls RecordBatch on an already-terminal batch, which refuses before auditing, so it cannot fail and proves nothing
  - id: F3
    severity: MEDIUM
    location: _lyx/plan/15-run-exit-dispositions.md
    summary: No card tests a batch-level warning reaching summary.md's Audit warnings section, which the first must-cover scenario requires
  - id: F4
    severity: MEDIUM
    location: _lyx/plan/08-record-batch-audit-dispositions.md
    summary: Step 3 fails the batch on a correctness finding with no report or a FAILED report, beyond D6's OK-report trigger, without stating why and without a test for either case
  - id: F5
    severity: MEDIUM
    location: _lyx/plan/26-loom-refusal-sweep.md
    summary: Preflight ways forward are wrong for fabric-sync (reconcile does not realign the weft branch) and for the weft half of worktree-clean
  - id: F6
    severity: MEDIUM
    location: _lyx/plan/24-webstercli-refusal-sweep.md
    summary: The fabric-sync failure fixture makes a push fail, but webstercli's fabricSync only commits, so the fixture cannot reach the rows
  - id: F7
    severity: MEDIUM
    location: _lyx/plan/26-loom-refusal-sweep.md
    summary: lyx loom commit-records' commit and push failures are reachable at every driver stop, name no way forward, and no sweep card lists or groups them
  - id: F8
    severity: MEDIUM
    location: _lyx/plan/26-loom-refusal-sweep.md
    summary: seed-missing and the decode branch of seed-incoherent cannot be reached through Shed, yet card 26 requires tests reaching them from a realistic state
  - id: F9
    severity: MEDIUM
    location: _lyx/plan/22-loom-goto-coherence.md
    summary: contracts/specs/loom-status-spec.md still says history outcomes are done, stuck or awaiting and that the budget reason is fixed; no card updates it
  - id: F10
    severity: LOW
    location: _lyx/plan/08-record-batch-audit-dispositions.md
    summary: A policy finding first seen on a no-report call is warned with no verify re-run, and the later OK report never re-runs verify for it
  - id: F11
    severity: LOW
    location: _lyx/plan/01-audit-severity.md
    summary: ClassifyViolation re-derives the _lyx dir through Dir(geom.AnchorRoot) instead of the told WebsterDir, and the per-named-spawn Key needs CheckParent to split its aggregate violation, which the card does not say
  - id: F12
    severity: LOW
    location: _lyx/plan/07-batch-fail-primitive.md
    summary: The ImpactSummary claim about readers is inexact (completedCards counts a failed batch as completed), and the BatchState Terminal and Status comments that list statuses go stale
  - id: F13
    severity: LOW
    location: _lyx/plan/23-websterengine-refusal-sweep.md
    summary: The unknown-batch-number way forward is scoped to BeginBatch, but the text comes from the shared findBatch that record-batch, await-batch and recover-batch also reach
  - id: F14
    severity: LOW
    location: _lyx/plan/04-batch-card-record.md
    summary: Card 4 changes recoverSpawn but defers its only assertion to card 11, so it cannot be tested on its own
  - id: F15
    severity: LOW
    location: _lyx/plan/26-loom-refusal-sweep.md
    summary: Card 26 spans five packages, mixing loom CLI verbs with gate-row producers, and is not one reviewable unit
---

### [BLOCKING] Batch-level audit warnings are lost on re-begin and recovery

**Location:** `_lyx/plan/02-audit-ledger.md`, with `_lyx/plan/08-record-batch-audit-dispositions.md` step 5 and `_lyx/plan/10-post-batch-failures.md`.

**Issue:**
`BeginBatch` (`internal/websterengine/beginbatch.go`) writes `deps.State.Batches[number] = &BatchState{…}`, a fresh record that inherits only `StartSHA`.
`recoverSpawn` (`recoverbatch.go`) returns a fresh `BatchState` that `RecoverSpawnOrAttach` assigns over the prior record.
Card 2 puts batch warnings in `BatchState.AuditWarnings` and the identities in `State.AuditDispositions`.
Two realistic flows hit the gap:
card 8 step 5 records a warning on a no-report call, and the ladder's next rung is a re-fork through `begin-batch NN`;
card 10's failed batch can carry warnings recorded in step 7, and its way forward is `recover-batch NN`.
Either way the new record drops `AuditWarnings`, while the disposition stays, so no later call ever records the finding again.
The warning disappears from `state.json`, and `RecordedAuditWarnings` never feeds it to `summary.md`.
That breaks D5's rule that each warning is kept in `state.json` once per identity and appended to `summary.md`.

**Fix:**
Card 2 also edits `BeginBatch` and `recoverSpawn` so the new record inherits the prior record's `AuditWarnings`, the same way `StartSHA` is inherited, and says why.
Add a test that records a warning on a no-report call, re-begins the batch, and asserts the warning is still on the record.

### [MEDIUM] Card 8's no-duplicate test is vacuous

**Location:** `_lyx/plan/08-record-batch-audit-dispositions.md`, first test bullet.

**Issue:**
"A second `RecordBatch` call on the now-terminal batch adds no second warning" never reaches the audit.
`RecordBatch` refuses a terminal record before auditing: today through `ErrNoBeginRecord`, after card 9 through the terminal-record refusal.
The test passes even if the ledger does no deduplication.
The retry that can duplicate a warning is one that follows a persisted non-terminal call, such as a no-report call followed by the re-forked report.

**Fix:**
Replace that assertion with the reachable retry:
a parent `fabric-reference` present on a no-report call (warning recorded and saved), then `BeginBatch` re-begins the batch, a fork writes an OK report, and `RecordBatch` records `done` with exactly one warning for that identity in state.
With F1 fixed, this test also covers the inheritance.

### [MEDIUM] No test carries a batch-level warning into summary.md

**Location:** `_lyx/plan/15-run-exit-dispositions.md`, tests.

**Issue:**
The first must-cover scenario requires a record-batch `fabric-reference` warning to appear in `summary.md`'s `## Audit warnings`.
Card 8 asserts the warning on `RecordResult.Warnings` and `bs.AuditWarnings`.
Card 14 unit-tests the appender.
Card 15 asserts `summary.md` only for the run-level integration-fork warning.
No test follows a batch-level warning through `Run`'s exit into `summary.md`.

**Fix:**
In card 15's named-spawn end-to-end test, also assert that `summary.md` has an `## Audit warnings` section carrying the batch-2 warning exactly once.

### [MEDIUM] Step 3 fails batches D6 does not name, with no rationale and no test

**Location:** `_lyx/plan/08-record-batch-audit-dispositions.md`, step 3 ("whatever the report says, and with no report at all").

**Issue:**
D6's trigger is record-batch rejecting an `OK` report on its merits.
Step 3 also fails a batch whose report is `FAILED` or absent.
The decision record does license this, but only indirectly: D2 says a correctness halt halts with a way forward, and D8 says each finding is dispositioned once.
Returning `NoReport` would drop a correctness finding, and only failing the batch satisfies both rules.
The ring does not re-form either.
If a still-running fork later writes its report to the live path, `recover-batch NN` passes `refuseRecoveringDoneReport` (card 11's `failed` exception), and `recoverSpawn` archives the report before spawning.
`begin-batch` and `record-batch` over that state name `record-batch` and `recover-batch` respectively (card 9), never each other in a loop.
The card states none of this reasoning and tests neither case.

**Fix:**
Add one sentence to step 3 grounding the extension in D2 and D8.
Add tests:
a correctness parent finding on a no-report call leaves the batch terminal `failed` with no report archived;
an OK report that lands afterwards is archived by `RecoverSpawnOrAttach`, which spawns.

### [MEDIUM] Preflight ways forward are wrong for fabric-sync and for a dirty weft

**Location:** `_lyx/plan/26-loom-refusal-sweep.md`, the preflight bullet.

**Issue:**
`fabric-sync` is `fabricengine.CauseBranchMismatch`: the weft worktree is not on the paired weft branch.
`lyx fabric reconcile` has no branch-realignment action; its actions are weft recreated, junction repointed, raw adopted, unmanaged reported, portal restored and stale removed.
`lyx fabric checkout` with no argument is the verb that re-syncs the weft side.
`worktree-clean` comes from `fabricengine.Clean`, which also reports "uncommitted state changes under `_lyx`" on the weft.
Committing that with git on the warp is not possible, and D1 allows git only when no `lyx` verb covers the case (`lyx fabric commit` does).
The card's own "take the way forward, then the row proceeds" test would fail for `fabric-sync`.

**Fix:**
`fabric-sync` → `lyx fabric checkout`, then re-step.
`worktree-clean` → commit or stash the warp's changes with git and commit the weft's with `lyx fabric commit`, then re-step.
`fabric-ready` and `junction` stay on `lyx fabric reconcile`.

### [MEDIUM] The fabric-sync failure fixture cannot reach its rows

**Location:** `_lyx/plan/24-webstercli-refusal-sweep.md`, tests.

**Issue:**
The tests use "a hub fixture whose fabric push is made to fail".
`internal/webstercli/sync.go`'s `fabricSync` only calls `f.Commit` on the scoped `_lyx` pathspec and never pushes, so a failing push leaves every `… but the fabric sync failed` row unreached.

**Fix:**
Make the weft commit fail instead: for example, a `hubforge` hub with a stale `index.lock` in the weft repository, or a failing weft hook.
Remove the obstruction, take the named way forward (`lyx fabric commit`, or the next bracket verb), and assert the state is committed.

### [MEDIUM] Sweep misses lyx loom commit-records' failures

**Location:** `_lyx/plan/26-loom-refusal-sweep.md`.

**Issue:**
`internal/loomcli/commitrecords.go` refuses with `loom: commit-records: commit failed: …` and `loom: commit-records: the commit landed locally but was not pushed: …`.
The ly-drive driver runs this verb at every stop and park, so a network or auth failure reaches it.
Neither message names a way forward, and card 26 neither fixes nor groups them.
The decision record's open risk says a missed refusal keeps its current behaviour.
Two smaller gaps are also unclassified:
approve's GitHub lookup and origin-URL failures are neither fixed nor grouped;
the `validate-*` verbs' "not yet valid" envelopes are a lint verdict, and the table should say so rather than stay silent.

**Fix:**
Add `internal/loomcli/commitrecords.go` to card 26:
commit failure → transient; re-run `lyx loom commit-records`;
push failure → `lyx fabric push`, or re-run `lyx loom commit-records`.
Test through `commitRecordsVerb`'s injected `commitStatusDeps`.
Group approve's lookup failures as a transient row, and state that `validate-*` verdicts are verb output, not refusals.

### [MEDIUM] Card 26's seed rows cannot be reached through Shed

**Location:** `_lyx/plan/26-loom-refusal-sweep.md`, the `loomengine.CheckSeed` bullet and tests.

**Issue:**
`CheckSeed` has one caller, the `Loom-Preflight` row (`internal/loomshed/loompreflight.go`).
`internal/loomengine/seed.go` documents that step 1's read gate pre-empts `seed-missing` and the decode branch of `seed-incoherent`, and `report.go` marks both "Unreachable through Shed".
D2 files such refusals as grouped wiring rows with no per-row test.
Yet card 26 requires each fixed row to be reached "from a status file as Shed writes it".
The shed step-1 refusal that pre-empts them is card 25's row.
Also, "`seed-incoherent` → seed a new run" must not attach to `CheckHalfFinished`, whose way forward card 22 sets to `goto`.

**Fix:**
Move `seed-missing` and the pre-empted `seed-incoherent` branches to card 26's grouped list, pointing at card 25's step-1 row.
Keep a way forward and test only for the reachable `seed-incoherent` branches (for example an out-of-vocabulary history outcome).
State that `CheckHalfFinished` is card 22's.

### [MEDIUM] loom-status-spec.md contradicts the goto outcome and the suffixed budget reason

**Location:** `_lyx/plan/22-loom-goto-coherence.md`, with `_lyx/plan/19-shed-step-way-forward.md`.

**Issue:**
`contracts/specs/loom-status-spec.md` says "Every `shed.history[].outcome` must be `"done"`, `"stuck"` or `"awaiting"`", and describes a blocked halt's error as "the fixed "bounce budget exhausted"".
The decision record's constraint says `goto` changes the status-file history contract, and card 19 appends a suffix to that reason.
No card edits the spec.
`internal/shedengine/run.go`'s `appendHistory` comment, "whose whole vocabulary is done, stuck and awaiting", goes stale the same way, and card 17 edits that file without updating it.

**Fix:**
Card 22 gains a `Prosa` target `contracts/specs/loom-status-spec.md`:
add `goto` as a history-only outcome written by `lyx shed goto`, and say the budget reason starts with `bounce budget exhausted` followed by its way forward.
Card 17 updates the `appendHistory` comment.

### [LOW] A policy finding first seen with no report skips the verify re-run

**Location:** `_lyx/plan/08-record-batch-audit-dispositions.md`, steps 5 and 7.

**Issue:**
Step 5 records a no-report call's policy findings as warnings and dispositions them.
When the re-forked fork's OK report arrives, step 1 drops those identities, so step 7 finds no policy findings and never re-runs the cards' verify commands.
The batch records `done` on a warning D5 would otherwise have backed with evidence.

**Fix:**
Step 7 runs `rerunCardVerifies` when this call has policy findings or the batch record already carries `AuditWarnings` (kept across re-begin by F1's fix).
A failing re-run fails the batch naming those warnings too.

### [LOW] Card 1's _lyx derivation and named-spawn keys

**Location:** `_lyx/plan/01-audit-severity.md`.

**Issue:**
`filepath.Dir(Dir(geom.AnchorRoot))` calls websterengine's path constructor on a told root inside the engine.
No websterengine engine code does that today; the Told-Geometry constraint the decision record names means the engine reads told fields.
The result is right in both modes: in standalone mode `AnchorRoot` is the state directory, and its `_lyx` holds the run's state, reports and plan.
`filepath.Dir(geom.WebsterDir)` reaches the same directory from a told field.
Separately, `CheckParent` emits one aggregate `named-spawn` violation for all `NamedSpawns`.
A per-spawn ordinal 1..`NamedSpawns` only works if `CheckParent` emits one violation per spawn, and the card leaves that change implicit.

**Fix:**
Use `filepath.Dir(geom.WebsterDir)`.
State that `CheckParent` emits one `named-spawn` violation per spawn, ordinal 1..`NamedSpawns`, so a new spawn gets a new identity and an old one keeps its own.

### [LOW] Card 7's reader claim and stale status comments

**Location:** `_lyx/plan/07-batch-fail-primitive.md`, ImpactSummary.

**Issue:**
"existing readers treat an unknown non-done status as not done" is not exact.
`verifyEveryBatchDone` does count `failed` as not done.
`completedCards` counts every `Terminal` record as completed, so a `failed` batch's cards drop out of later drift scoping, as a `stuck` batch's do.
No reader validates the status set, so nothing rejects `failed`.
`BatchState.Terminal` and `BatchState.Status` in `state.go` document "done, stuck, or dead", and card 7 does not edit `state.go`.

**Fix:**
Reword the ImpactSummary to say a `failed` record is read exactly as a `stuck` one is: terminal for `completedCards`, not done for `verifyEveryBatchDone`, and printed verbatim by `lyx webster status`.
Add `state.go` to card 7's Edit list for the two comments.

### [LOW] Unknown-batch way forward should live in findBatch

**Location:** `_lyx/plan/23-websterengine-refusal-sweep.md`, "`BeginBatch`, batch number not in the plan's batches".

**Issue:**
The message comes from `findBatch` in `beginbatch.go`.
`RecordBatch`, `AwaitBatch`, `RecoverSpawnOrAttach` and `PersistRecoveryTerminal` return it too.
A fix scoped to `BeginBatch` leaves those verbs without the way forward.

**Fix:**
Put the way forward in `findBatch`'s message and name every verb that reaches it.

### [LOW] Card 4 is not testable on its own

**Location:** `_lyx/plan/04-batch-card-record.md`.

**Issue:**
Card 4 edits `recoverSpawn` to set `Cards` but defers the assertion to card 11.
`recoverbatch_test.go` already has a recovery fixture, so nothing forces the deferral.

**Fix:**
Add `internal/websterengine/recoverbatch_test.go` to card 4 with an assertion that a spawned recovery record carries `Cards`, and drop the deferral sentence.

### [LOW] Card 26 is too broad for one reviewable unit

**Location:** `_lyx/plan/26-loom-refusal-sweep.md`.

**Issue:**
The card edits `loomcli`, `loomengine`, `loomshed`, `preflightshed` and `shedadapters`: CLI verb refusals and gate-row Stuck reasons, with different fixtures (GitHub-backed approve, status-file seeds, hubforge preflight hubs).
Card 23 stays within one package and one module surface, so it is not split.

**Fix:**
Split card 26 into a loom CLI verb card (`approve`, `start`, `commit-records`) and a loom gate-row card (seed, preflight, batchifier, commit seams, Webster adapter).

## Focus settlements without a finding

- Card 10's premises hold.
  Card 8's step 4 advances attribution before the report check, so transcripts are consumed by the time `postBatchChecks` runs, and D6's "marks the offending fork transcripts as seen" holds on the `ErrCardNotDone` path.
  `BindHandles` receives `in.Cards`, so bind findings concern the batch's own cards.
  Drift runs over `PendingPlan(plan, completedCards(…, batchNumber))`, which excludes this batch and every terminal one, so its blocking findings concern cards still to be built.
- Card 8's caller claim holds: `internal/webstercli/recordbatch.go` is the only production caller of `websterengine.RecordBatch`.
- Card 15's adapter claim holds: `shedadapters/webster.go` maps `stuck` to `shedengine.Stuck` with `StuckReason` and `done` to Done.
  `lyx webster run` prints `result.Warnings`, and its post-run `fabricSync` commits `_lyx`, so run-level warnings in `state.json` are committed through webstercli with no new card.
- Card 19's Go-side claim holds: `loomengine/anomaly.go` is the only code comparing against `ReasonBounceBudgetExhausted`; F9 covers the spec.
- Card 20's claim holds: all three subtrees destructure `verbs[0..3]` by position, and the two tests that range over `Verbs` look commands up by name.
- Card 23's claims hold: `Run` archives a stale `outcome.yaml` at entry, and `refuseRecoveringDoneReport` passes an unparseable report, so neither needs a control-flow change.
- Card 26's anomaly claim holds: loom's anomaly detection reads state and the budget reason only.
