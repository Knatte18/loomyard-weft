---
dispositions:
  - id: F3
    disposition: fixed
  - id: F10
    disposition: fixed
  - id: F11
    disposition: fixed
---

## Changes

- **F3** (`driftengine`, two commits):
  - Deleted the stored old-Drift readers.
    - `scopeText` no longer renders a `partial` scope, so a stored one reads as `full`.
    - `classifyCurrent` and `settleUnchangeable` no longer take a leave-out list, so `skippedCases` and `testFiles` are gone.
    - `DiffCaptures` lost its `leaveOut` parameter, since nothing passes one any more.
    - `oldCheckLines` and `compareText` are gone. A record with no last check and no deferral renders `- Check: not run.`
    - `ScopePartial`, the `Acceptance` type, the `AcceptedBy*` and `Ruling(Un)Ruled` constants, and the `ScopeRecord` fields only the old Drift read are gone.
      Those fields are `Skip`, `Accepted`, `Withdrawn`, `HaltNamed`, `HaltNamedAt`, `GateHaltAt`, `ComparedHead`, `ComparedAt`, `ComparePassed` and `AcceptedProfileHead`.
    - The waiver fields stay, since `skipReason` and `verify.go` still read them.
      `BaselineHead` stays, since `expect.go` reads it.
    - `accepted.go` held only `copyFile` once `testFiles` went, so it is now `copyfile.go`.
  - Dropped the wording about old partial records in `recordScope`'s comment and in the `doc.go` lines the review named, and the partial-to-full Warn sentence at `doc.go:469`.
  - Tests: removed the rows that pinned the deleted readers (old partial verdict, old compare and waiver verdicts, old partial check, old recorded partial, `LeaveOutCaseYieldsNoCell`).
    The await row that seeded a `partial` scope now seeds `full`.
    The records-guard unguarded-read row now reads a record carrying the legacy `skip` and `accepted` keys, which pins the tolerant read.
  - Follow-up commit: `CaptureRecord.Skip` had no reader or writer left, so it and its `doc.go` mention are gone (consistency lens, below).
- **F10** (`batcher`): `retiredKeyError`'s default-layer arm now offers `lym config reconcile --layer default --apply` from the hub's prime worktree instead of `lym fabric reconcile`.
  The `config.go` comment, the `doc.go` sentences and the default-layer row of `config_test.go` follow.
- **F11** (`configsync`): `TestReconcileHubWideAt_SeedsFromHubPrimeOrTemplate` gains a `withTemplateBase` fixture field, a `wantAddedHas` assertion, and a models row.
  The row has an absent hub file, a prime copy holding only an operator alias, and the module's template written as the template base.
  It asserts the template alias `sonnet` is in `Added` and in the written file, which fails once `!fileAbsent` leaves `configsync.go:482`.

Gate repair: the gate found two wrapped comment lines from F10, in `internal/batcher/config.go` and `internal/batcher/doc.go`.
Each is now two sentences, one per line, with no sentence broken mid-way.

Verification: `lym gate test` over `internal/driftengine`, `internal/batcher` and `internal/configsync` passes at tier0, and `go build` and `go vet` are clean.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency, LOW: `CaptureRecord.Skip` and its `doc.go` mention left over from the deleted skip list. Fixed as proposed, in its own commit.
- fix-consistency, LOW: `_lym/landing/summary.md` does not say the stored old-Drift readers are deleted or that the waiver readers stay. Deferred, see below.
- fix-proposal: no findings.
- fix-affected-set: no findings. It noted the same `CaptureRecord.Skip` leftover as the consistency lens, merged into that finding.

## Disputed

None.

## Deferred

- The `_lym/landing/summary.md` update the consistency lens raised.
  The file sits under `_lym`, which this round never writes or commits, and the loop that spawned the round owns it.
  The summary's "Kept on purpose" list needs one line naming the waiver readers in `skipReason` and `verify.go`, which stay, and a line saying the stored old-Drift scope readers are deleted.
  The parent should add it.

## Rulings

None.
