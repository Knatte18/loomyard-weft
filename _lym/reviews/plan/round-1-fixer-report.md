# Fixer report — plan, round 1

Every finding's premise held against the code; every finding is fixed.

## Changes

- **F1** — fixed differently on one detail.
  Card 2 creates `WeftPathsChangedSince` over `Repo.ChangedFilesSince`, with a `TestCheckoutStates` row.
  Card 15 adds the `WeftChangedSince` and `LoomWeftDirs` Deps fields and the exempt rule in `Changes`, with a `TestJudgeSnapshot` row.
  Card 16 adds the `TestDriftVerify` resume row.
  Not as proposed: the review asked for anchor-relative paths.
  The new function returns paths relative to the weft worktree's root instead, the form `CheckoutState`'s lists and `Repo.ChangedFilesSince` use, so `Changes` compares one form.
  The exempt directories are joined under `AnchorRel` to match, since the weft holds the anchor subpath (`internal/fabricengine/commitweftpaths.go:101`).
  The self-check widened the exempt set to include loom's friction directory (see Self-check).
- **F2** — fixed as proposed.
  Card 16 no longer deletes `captureForAcceptance`, `agentAccept` or `sessionAccept` and leaves them in place.
  Card 25 deletes all three.
- **F3** — fixed as proposed, plus one member.
  Card 4 edits `templateConfig`, `newVerifyFixture` and `TestDriftVerify`.
  Card 26 edits `templateConfig`, and also `TestSeatTables`, which sets `VerifyMaxRuns` at `seats_test.go:53` and `:142`.
  Card 16 edits `minimalEntries` to use the `judgement` gate.
- **F4** — fixed as proposed.
  Card 19 wraps the parser refusal so it names a way forward, and `TestAmend` asserts it.
- **F5** — fixed as proposed.
  Card 10 adds a new step 3, a `PastRow` skip before the baseline checks; the later steps renumber.
  Its test row leaves `awaited_at` unset.
  Card 14's Drift-Await row seeds no `awaited_at`.
- **F6** — fixed as proposed.
  Card 8: `Unguarded` now skips only the digest compare, and a missing directory still faults; a new test row covers this.
  Card 12's missing-baseline row runs with no `Drift-Baseline` commit.
- **F7** — fixed as proposed.
  Card 18's header note now names no removed verb.
- **F8** — fixed as proposed.
  Card 9 sets the cadence: one second, doubling to thirty.
  Card 9 also adds `PATTERN-daemon-wakeups.md` to its Edit list, with the line it adds.
- **F9** — fixed as proposed.
  Card 13 moves `internal/loomrecipe/doc.go` to Edit, adding the row orders.
  Card 16 also edits that doc, for the Drift-Recheck sentence.
  Card 22 edits `docs/overview.md`, covering the verb line and the drift paragraph.
- **F10** — fixed as proposed.
  Card 19's held-lock refusal names waiting for the check, then defer again.
  `TestDeferWaive` gains a row for it.
  The self-check also restored `lym loom resume` to this refusal (see Self-check).
- **F11** — fixed as proposed.
  Card 25 deletes `CountedCapture` and `TestCountedCapture` and drops its doc.go line.

## Self-check

- fix-claims, BLOCKING — the F1 exempt set missed loom's friction notes. Fixed with my own change, not the lens's first suggestion of exempting every path loom's record commits write.
  Card 15's field is now `LoomWeftDirs`: the run's shed directory plus `loomengine.LoomFrictionDirRel`.
  It gains a `TestJudgeSnapshot` friction-note row.
  Card 16's resume row covers a halt note.
- fix-proposal, MEDIUM — the same friction-directory gap: duplicate of the above, fixed there.
- fix-affected-set, MEDIUM — the same friction-directory gap: duplicate of the above, fixed there.
- fix-consistency, MEDIUM — card 16's `driftVerifyEntry` seam list lacked `WeftChangedSince`: fixed as proposed.
  The list gains it, and `TestDriftEntries` covers the missing seam.
- fix-consistency, LOW — card 10's `awaited_at` row named no status row: fixed as proposed.
  The row now runs at a row past Drift-Expect.
- fix-proposal, LOW — after F6, a scope-`off` run would admit `lym drift defer` at any blocked writer: fixed with my own change.
  Card 19 admits the `CheckBaseline` route only on a scope record whose scope is not `off` and that holds no deferral.
  It gains a row refused at a blocked Darn-Write on a scope-`off` run.
- fix-proposal, LOW — F10's refusal dropped `lym loom resume` for a running main capture: fixed with my own change.
  The refusal names both ways forward.

## Disputed

None.

## Deferred

None.
