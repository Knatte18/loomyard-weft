---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/12-reset-start-archives.md
    summary: "Card 12 puts gitrepo.New(worktree).ResetKeep in internal/websterengine/reset.go, which the PATTERN-fabric-git guard test cmd/lyx/rawgitmutation_test.go bans there and the decision record's webster runs no raw git constraint forbids"
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/plan/12-reset-start-archives.md
    summary: "Cards 11, 12 and 14 do not name the refusal-spec rows they rewrite or delete, so stale reset rows and the toward the --fresh route phrasing survive; the same method gap as round 1 F7"
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/plan/12-reset-start-archives.md
    summary: "Reset to start now archives pending audit findings and uncheckable batch records with none of the run --fresh pending-findings guards, and the card states no bound for that"
  - id: F4
    severity: MEDIUM
    class: scope
    location: _lyx/plan/20-plan-write-archives-webster.md
    summary: "Card 20 moves the Webster run record out of its tracked directory but leaves the Plan-Write commit seam on the plan directory only, so the moved-from deletions are not staged"
  - id: F5
    severity: MEDIUM
    class: design
    location: _lyx/plan/14-retire-fresh-after-reset.md
    summary: "Cards still miss sites a grep finds, the gap round 1 F4, F5, F11 and F13 already filed per item; here cli_test.go asserts the reworded --fresh text, persistPlanFingerprintRebaseline copies the plan hashes, and the webster group help lists the old reset targets"
  - id: F6
    severity: LOW
    class: design
    location: _lyx/plan/12-reset-start-archives.md
    summary: "Card 12 asserts keep-reset git semantics already pinned by card 10 TestResetKeep, and asserts the archive at the engine level where TestResetCmd owns it, in the read-only PlanReset test file"
  - id: F7
    severity: LOW
    class: design
    location: _lyx/plan/11-reset-targets.md
    summary: "Card 11 tests the state-only report-head refusals in the integration-tier TestPlanReset although Geometry.Git is fakeable and the untagged reset_test.go already holds the pre-git refusals"
  - id: F8
    severity: LOW
    class: design
    location: _lyx/plan/20-plan-write-archives-webster.md
    summary: "The plan-write archive collision row cannot be reached from on-disk state because the rotation always hands ArchiveRunRecord a fresh archive directory"
  - id: F9
    severity: LOW
    class: design
    location: _lyx/plan/08-computed-merriam-base.md
    summary: "Card 8 derives the Go constant from the Master stencil at its own position, but cards 13 and 17 edit that stencil afterwards, so the computed base no longer reproduces the measured start at landing"
  - id: F10
    severity: LOW
    class: scope
    location: _lyx/plan/02-batcher-orientation.md
    summary: "Card 2 adds a Breakdown.Base field that no card reads and the decision record does not call for"
  - id: F11
    severity: LOW
    class: consistency
    location: _lyx/plan/06-webster-config-invalid.md
    summary: "Card 1 marks every module's content failures with ErrInvalid, so a pre-run errors.Is test would also flag shuttle, reed and models failures, against card 6's bound to batcher.yaml and webster.yaml"
---

### [BLOCKING] Card 12's standalone keep-reset violates the raw-git guard

**Location:** `_lyx/plan/12-reset-start-archives.md` (Standalone paragraph); guard at `cmd/lyx/rawgitmutation_test.go`.

**Issue:**
Card 12 has `resetCmd` call "a new websterengine function in `reset.go` that performs the planned move through `gitrepo.New(worktree).ResetKeep(sha)`", and says `reset.go`'s file header will "name it as the standalone exception".
`TestNoRawGitMutation_WebsterProductionSource` fails any non-test file under `internal/websterengine` that contains the token `gitrepo.New(` unless the file is on `rawGitMutationAllowlist`.
Only `gitwrap.go` is on it, and the entry's reason is "grandfathered read-only exemptions".
`reset.go` is not on it and card 12 does not list the test file, so the card's own gate fails.
The decision record's constraint is the same rule stated as intent: "the standalone keep-reset is a `gitrepo` method on the `gitexec` side; webster runs no raw git" (`PATTERN-fabric-git`, `PATTERN-gitrepo-client-boundary`).
A header comment that declares an exception does not make one.
This is a regression from round 1: the fixer's report (F10) dropped the `gitwrap.go` wrapper and had card 12 call `gitrepo.Repo.ResetKeep` from `reset.go` directly, to avoid two gate overlaps.

**Fix:**
Choose one route and state it in card 12.
Either put the standalone move in `internal/webstercli`, which the guard does not scan, behind a told seam, or add a mutating method to the `websterengine.Git` seam whose real implementation sits in the allowlisted `gitwrap.go`.
The second route needs `cmd/lyx/rawgitmutation_test.go` listed in the card with a widened allowlist reason, and the `PATTERN-fabric-git` "known gap" wording amended in the same commit.
Drop the "file header names the exception" sentence either way.

### [MEDIUM] Refusal-spec rows are not named, so stale rows survive

**Location:** `_lyx/plan/12-reset-start-archives.md`, `_lyx/plan/14-retire-fresh-after-reset.md`, `_lyx/plan/11-reset-targets.md`.

**Issue:**
Round 1 F7 filed this gap for cards 11, 6 and 16, and the fix named rows only in those cards.
Cards 5, 6, 7, 13, 16, 17 and 19 now name the spec rows they rewrite or delete.
Cards 12 and 14 do not.
Card 12 says only "drops the start rows the archive-only path replaces and the standalone refusal row".
Card 14 says only "webster rows naming the pair are rewritten".
Rows in `contracts/specs/refusal-spec.md` that stay stale under that wording:
- `reset: no recorded target, start` (way forward `run --fresh`): card 12 makes this an archive-only case, so the row has no refusal left.
- `reset: target commit missing` and `reset: target not an ancestor of HEAD`: the second still says "for `start`, run `lyx webster run --fresh`", which card 12 makes unreachable for `start`.
  Both keep applying to `pre-fix` and the new targets, so they need rewording, not deletion.
- `reset: dirty tracked path the run did not write`: card 12 skips it in standalone mode and drops it on the archive-only path, and the row says neither.
- `fresh: recorded starts diverge`: its cell says "the reset resolving the commit every recorded start descends from", which card 12 turns into an archive-only reset.
- `audit: fork-state-write`, `audit: parent-write, tracked, ...` and `audit: fabric-reference`: each says "refuses toward the `--fresh` route", a phrase card 14 never mentions because it is not the literal pair.
- Rows carrying the pair that card 14 must rewrite but does not list: `rebaseline: failed batch recovery cannot check`, `restore-plan: copy missing`, `card not done: delete still referenced...`, `run-exit audit correctness`, `recover-batch: delete still referenced...`, `accept-audit: finding not checkable`, `fresh over a differing suspect path`, `fresh: HEAD not at the start commit`.
  Rows `rebaseline card set changed`, `rebaseline: begun card of a done batch`, `rebaseline overview changed`, `plan fingerprint mismatch`, `plan changed before the verb's own rewrite`, `accept-audit --batch: not acceptable` and `recover-batch: findings recovery cannot check` are touched by card 14 and by 13, 16 or 19, so the order of the two rewrites also needs to be stated.
Card 14's retired-steps test scans `contracts/specs/`, so an unlisted row fails the test instead of being fixed on purpose.

**Fix:**
List each row by its name in the Intent of cards 11, 12 and 14, with delete or reword per row, as cards 13 and 19 do.
Add the `--fresh` route phrasing in the three audit rows to card 14.

### [MEDIUM] Reset to start bypasses the pending-findings guard without a stated bound

**Location:** `_lyx/plan/12-reset-start-archives.md` (Bound line), `_lyx/plan/14-retire-fresh-after-reset.md`.

**Issue:**
`run --fresh` refuses through `freshPendingDrop` before it drops `State.PendingAuditFindings` or a batch record's `Uncheckable` entries: a contract file a fork wrote last, a suspect path still differing from the start commit, HEAD not at the start commit, a plan path differing from the recorded plan.
The refusal-spec header states the invariant: "a correctness halt clears only on evidence ... never on an acknowledgement".
Card 12 makes `reset --to start` archive `state.json` after the move.
In a pair, the move puts HEAD on the start commit and discards the run's tracked changes, but the archive-only path and the standalone keep-reset (D4) move nothing or carry tracked changes across.
Findings are archived there with none of those checks, and card 14 rewrites the `freshPendingDrop` refusals so their way forward is that same reset.
The guard those refusals protect is bypassed by the way forward they name.
Card 12's Bound lists only "a branch rewritten under the run and any uncommitted changes".
D3 retires the branch-move refusals; it does not mention the pending-findings guards.

**Fix:**
State in card 12's Intent what happens to pending findings and uncheckable records on every reset-to-start path.
Either keep an equivalent check on the paths that move nothing (the `freshPendingDrop` conditions, run before the archive), or state that the archive drops them, with the bound, and have card 14 reword the `freshPendingDrop` rows to match.

### [MEDIUM] Card 20 leaves the moved-from Webster deletions unstaged

**Location:** `_lyx/plan/20-plan-write-archives-webster.md`; `internal/loomcli/wiring.go` (`CommitPlan`, `reworkCommitPathspec`).

**Issue:**
Card 20 archives the run record into `<archive-dir>/webster` inside the plan directory, so the additions are committed by `CommitPlan`, whose pathspec is the plan directory only.
The record's source, webster's durable directory, is tracked: `CommitWebster` commits `websterengine.DirRel()`.
PR-Rework needed `reworkCommitPathspec` to add that directory for exactly this reason: its comment says "so the moved-from deletions and the archive land in one commit".
Card 20 targets neither `internal/loomcli/wiring.go` nor a pathspec helper, so after a rotation the deletions sit unstaged in the records worktree until the Webster row's own commit.
The Plan-Review, Plan-Bouncer and later rows run in between.

**Fix:**
Add the `CommitPlan` pathspec (or a helper beside `reworkCommitPathspec`) to card 20, including webster's directory when a run record was archived, with a test row.
Alternatively state why the deletions may wait for `CommitWebster`.

### [MEDIUM] Method gap: cards miss sites that a grep of the changed text or structure finds

**Location:** `_lyx/plan/14-retire-fresh-after-reset.md`, `_lyx/plan/19-rebaseline-card-index.md`, `_lyx/plan/11-reset-targets.md`.

**Issue:**
Round 1 filed F4, F5, F11 and F13 as individual missed callers, tests and docs.
The same gap recurs, so it is filed once as a method finding, with these instances:
- `internal/webstercli/cli_test.go` `TestRebaselineCmd_Refusals` asserts `--fresh` in two rows, "removed card of a begun batch" and "named card of a done batch".
  Card 14 rewords the first through `freshRestartSteps` in `rebaselineBatches`, and card 19 rewrites the second.
  Neither lists `cli_test.go` (card 2 lists it only for a `Weights` literal).
  The plan template's rule 4 requires every test asserting reworded text to join the rewording card, so the assertion fails under that card's own gate.
- `persistPlanFingerprintRebaseline` in `internal/webstercli/cli.go` assigns `fresh.PlanFingerprint` and `fresh.PlanFileHashes` onto a reloaded state.
  D7 and card 19 record the frame hash "wherever `PlanFileHashes` is" restamped, and name `restampBaseline` and `Run`'s branches, so a failed bracket verb that rewrote the plan drops the new field.
  `TestPersistPlanFingerprintRebaseline` in `cli_test.go` is the existing test for it.
- The `Verbs:` list in `Command`'s `Long` in `internal/webstercli/cli.go` still reads `lyx webster reset --to start|pre-fix ... move the task branch back to the run's start commit or the verify gate's pre-fix head`.
  The decision record's scope includes webstercli help text, and card 11 rewrites only `resetCmd`'s own help.

**Fix:**
Add `internal/webstercli/cli_test.go` to cards 14 and 19 (the two rows, and `TestPersistPlanFingerprintRebaseline` for the frame hash), add `persistPlanFingerprintRebaseline` to card 19 or state why the hash cannot change in that path, and add the `Command` help in `cli.go` to card 11.
Each card writer should end with a grep of the old text, the changed struct field and the changed CLI surface, listing every hit.

### [LOW] Card 12 repeats keep-reset semantics and puts the archive check in the wrong test

**Location:** `_lyx/plan/12-reset-start-archives.md` (last test paragraph); `_lyx/plan/10-gitrepo-reset-keep.md`.

**Issue:**
Card 10's `TestResetKeep` already pins a move with an unrelated dirty file kept and a refusal over a dirty file the move would overwrite, leaving HEAD and the file untouched.
Card 12's `reset_integration_test.go` asserts the same two outcomes again for the standalone function; only the per-path way forward is new at that surface.
The same paragraph asks that file to assert "a moving reset to start archives the record and a following state load finds none".
`TestPlanReset` in that file is read-only, and a moving pair reset is performed by `resetCmd`, so `TestResetCmd` (`webstercli/reset_test.go`, step "start resets head and own dirt and keeps untracked") owns it; the archive itself is pinned by `archive_test.go`.

**Fix:**
Keep one assertion per behavior: the archive as an assertion in the existing `TestResetCmd` start step, the keep semantics in `TestResetKeep`, and only the per-path way forward and the `uncommitted` listing at the standalone surface.
Name which function `reset_integration_test.go` drives.

### [LOW] Card 11 tests fakeable refusals in the integration tier

**Location:** `_lyx/plan/11-reset-targets.md` (test list).

**Issue:**
The terminal-batch, no-report, later-begun-batch, live-strand, no-recorded-commit and later-recorded-start refusals read state, report files and the reed seam; the first git question, `refuseMidMerge`, goes through `Geometry.Git`, which tests already fake.
Card 11 puts them in the integration-tier `TestPlanReset`, while untagged `reset_test.go` already holds the pre-git refusals (`PATTERN-test-economy`: a behavior testable with a fake is tested untagged).

**Fix:**
Put those rows in `reset_test.go` with a fake `Git`, and keep only the ancestry and dirty-path rows in `TestPlanReset`.

### [LOW] The plan-write archive collision row is unreachable

**Location:** `_lyx/plan/20-plan-write-archives-webster.md`.

**Issue:**
`ArchiveRunRecord` reports a collision only when `<dest>/<entry>` already exists.
The rotation passes `<archive-dir>/webster` where `archive-dir` comes from `firstFreePlanArchivePath`, so the destination is never populated; a retry after a partial failure picks the next free `-N` directory.
The refusal-spec classes put a state "unreachable from any on-disk state a run can produce" in the grouped wiring-guard row with no per-row test, yet the card adds a row and a seam-error test for it.

**Fix:**
Drop the collision row and keep the busy row, or state which on-disk state reaches the collision.

### [LOW] The computed-base constant is fixed before the stencil stops changing

**Location:** `_lyx/plan/08-computed-merriam-base.md`.

**Issue:**
The constant is 20600 minus the computed size of `CLAUDE.md` and the Master stencil "at this task's HEAD", written into a comment as the derivation.
Cards 6 and 7 edit the stencil before card 8, but cards 13 and 17 add ladder rungs after it, so at landing the computed base is a few lines of stencil above 20.6K.
The record's auto-mode assumption says drift is absorbed by refitting `orientation`, so the harm is small; the card still claims the base "reproduces the measured start".

**Fix:**
State the derivation point in the card (the stencil as it stands when card 8 runs, with drift left to the refit), or place the constant after card 17.

### [LOW] Breakdown.Base has no reader

**Location:** `_lyx/plan/02-batcher-orientation.md`.

**Issue:**
Card 2 adds `Breakdown.Base` "so a recorded run can be fitted against starts minus its base".
Card 9 recomputes each run's base from the plan and the base commit and never reads `Breakdown.Base`.
D13 does not list the field, and nothing else consumes it.

**Fix:**
Drop the field, or have card 9 read it for runs that recorded it.

### [LOW] Card 6's config_invalid bound does not survive the sentinel's reach

**Location:** `_lyx/plan/06-webster-config-invalid.md`; `_lyx/plan/01-configengine-invalid-mark.md`.

**Issue:**
Card 1 marks the content failures of every module's `configengine.load`, so `shuttleengine.LoadConfig` and `reedengine.LoadConfig` errors carry `ErrInvalid` too.
Card 6 says `resolvePersistentPreRun` prints a content failure with `config_invalid: true` and that "the other modules' config loads in the wiring keep their plain refusals".
If the pre-run decides by `errors.Is(err, configengine.ErrInvalid)` on the error `wire` returns, a broken `shuttle.yaml` also gets the flag and stops Master, against D10's bound to the `batcher.yaml` and webster config loads.

**Fix:**
Say how the pre-run knows the refusal came from the builder, for example a typed refusal returned only by it, and add a test row for an invalid `shuttle.yaml` carrying no flag.

## Prior rounds

Round 1's other fixes hold: card 20 names its two loom rows (F1), card 7 and card 11 extend the existing tests (F2, F3), card 14 names the three call sites and keeps `PlanEditError`'s signature (F4), `StartFit.write` is in card 2 (F5), card 12 states the standalone dirty-path choice (F6), cards 11, 6 and 16 name their rows (F7), and F8 to F14 are present.
The round 1 F10 fix regressed into F1 above, and the recurring gap of round 1 F7 and F4/F5/F11/F13 is F2 and F5 above.
No deferred or disputed item exists.

## Focus coverage

1. Refusal-spec rows: every card that adds, rewords or removes a refusal names `contracts/specs/refusal-spec.md`; the stale or overlapping rows that survive are in F2 (cards 11, 12, 14) and F8 (the Plan-Write collision twin).
   Cards 5, 6, 7, 13, 16, 17, 19 and 20 name their rows and leave no stale twin.
2. Test economy: card 7 extends `TestBeginBatch_Refusals` and `TestBeginBatchCmd_ReportOnDisk`, card 4 extends `TestReconcileAll`, card 11 extends `TestPlanReset` and `TestResetCmd` and does not ask `PlanReset` to assert a HEAD move.
   Placement and duplication issues are F6 and F7.
3. Build order: every caller of `Weights.MasterBase`, `Batcher.Batch`, `PeakContext` and `NewLoomPreflight` is in card 2 or card 5 (`tools/tokencount`, `webstercli/cli_test.go` and the websterengine fakes included), and the `fingerprintMismatchWayForward` re-entry parameter is passed by its three call sites inside card 14, ahead of card 19.
   No finding.
4. Standalone reset: card 12 states that `PlanReset` skips its foreign-dirty-path refusal in standalone and keeps it in a pair, and its per-mode trees, keep-refusal way forward and tests follow that choice.
   The open point is where the standalone move runs (F1).
