---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: _lyx/plan/04-drop-model-assertion.md
    summary: Card 4 changes the initState signature but two other webstercli test files that call it are not in its Edit targets, so the plan's own go vet fails
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/07-delete-blocked-by-later-card.md
    summary: Card 7 rewords one stencil spot for a failed-batch-leads-to-recover-batch concept that other ladder and progress lines state unconditionally, and names no claims-pinned clause to keep
  - id: F3
    severity: MEDIUM
    class: decision
    location: _lyx/plan/07-delete-blocked-by-later-card.md
    summary: Card 7 leaves open which envelope the new recover-batch refusal carries, and internal/webstercli/recoverbatch.go that maps refusals to envelope flags is not an Edit target
  - id: F4
    severity: MEDIUM
    class: scope
    location: _lyx/plan/08-background-shell-outcome.md
    summary: Card 8 changes when RunResult.Warnings entries appear and their wording, but internal/websterengine/doc.go still states done-only warnings with the old text and is not an Edit target
  - id: F5
    severity: MEDIUM
    class: decision
    location: _lyx/plan/11-reset-moves-remote-branch.md
    summary: Card 11 skips the remote update only under SkipPush and is silent on SkipGit, while the reset test fixture sets WEFT_SKIP_GIT=1 on every test and the planned CLI rows need the update to run
  - id: F6
    severity: MEDIUM
    class: scope
    location: _lyx/plan/04-drop-model-assertion.md
    summary: Card 4 removes sandbox scenario W2 whole on the ground that the mechanism it armed is gone, but W2c checks fork-transcript flush timing that is unrelated to the model assertion and no decision licenses dropping it
  - id: F7
    severity: MEDIUM
    class: decision
    location: _lyx/plan/00-overview.md
    summary: The cli-drive Shared Decision justifies driving Publish below the CLI by the resumed half needing a GitHub fake, but the D8 acceptance item is only the push-rejected stuck reason, which halts before any GitHub call
  - id: F8
    severity: LOW
    class: consistency
    location: _lyx/plan/11-reset-moves-remote-branch.md
    summary: Card 11 adds a tenth executor, banned token and Kind but rewords only two of the tallies that pin nine executors, nine tokens and nineteen kinds in comments
  - id: F9
    severity: NIT
    class: consistency
    location: _lyx/plan/10-fabric-remote-only-commits.md
    summary: Card 10 passes HEAD to CommitsNotIn while card 9 requires both arguments to be valid hex object names
---

### [BLOCKING] Card 4 breaks two test files it does not list

**Location:** `_lyx/plan/04-drop-model-assertion.md` (Tests paragraph, last sentence).
**Issue:** The card says `initState` in `internal/webstercli/verbs_test.go` "loses its model parameter".
`initState(t, "master-model")` is also called from `internal/webstercli/acceptaudit_test.go` (line 23) and `internal/webstercli/restoreplan_test.go` (line 19).
Neither file is in the card's Edit targets, so following the card exactly leaves two test files that no longer compile, and the plan's own `go vet ./...` and `go test ./...` fail.
**Fix:** Add both files to card 4's Edit targets and name the call-site update in its Tests paragraph.

### [MEDIUM] Card 7's stencil reword leaves other ladder lines false and risks a pinned clause

**Location:** `_lyx/plan/07-delete-blocked-by-later-card.md` (Docs paragraph, the `webster-template-master.md` sentence).
**Issue:** The card rewords "where a failed batch leads to `recover-batch`" in one place.
The stencil states that concept unconditionally in several lines: the progress rung `failed → run recover-batch ... exactly as for stuck`, the ladder rung `record-batch refuses with {"batch_failed": true} → ... run recover-batch backgrounded, then follow the recover-batch rungs above`, and the stuck and no-report rungs.
`contracts/stencils/claims_test.go` pins the substring "run `lyx webster recover-batch <NN>` backgrounded, then follow the recover-batch rungs" and "- `failed` → webster rejected that batch's report", and the card names neither clause as one to keep.
A reword that drops either breaks the claims test, and one that keeps the lines unchanged leaves Master told to call `recover-batch` on a failure whose way forward is a plan edit.
**Fix:** State that the card searches the whole stencil for the recover-batch-is-the-way-forward concept and amends each line that stays false by adding the plan-edit exception, keeping the two pinned clauses verbatim.

### [MEDIUM] Card 7 does not say how the recover-batch refusal reaches Master

**Location:** `_lyx/plan/07-delete-blocked-by-later-card.md` (recover-batch paragraph and Edit targets).
**Issue:** `internal/webstercli/recoverbatch.go` maps only `ErrRecoveryNeedsFresh` to a flagged envelope (`needs_fresh`); every other refusal is a flagless `output.Err`.
Master's failure ladder keys each rung off an envelope flag, and `TestMasterStencilCoversEverySingleFlagRefusal` guards that.
The new D4 refusal as written is a plain error, so no ladder rung covers it, and the card lists no `webstercli` production file.
**Fix:** Decide in the card that the refusal wraps a sentinel the CLI maps to the existing `batch_failed` envelope flag (the rung that ends the run stuck quoting the message), add `internal/webstercli/recoverbatch.go` to the Edit targets, and have the cobra test assert the flag.

### [MEDIUM] Card 8 leaves `doc.go` describing the old warning

**Location:** `_lyx/plan/08-background-shell-outcome.md` (Edit targets).
**Issue:** `internal/websterengine/doc.go`, "Background shells in Master's wait", says the warning appears "On a done outcome", quotes the old warning text, and says "A non-done outcome keeps its own error or stuck reason".
The card makes the warning appear on done, stuck and paused outcomes with new wording, so that paragraph becomes false.
CLAUDE.md requires the module doc to change in the same commit as observable behavior.
**Fix:** Add `internal/websterengine/doc.go` to the Edit targets and say the paragraph is rewritten for the new warning scope and wording.

### [MEDIUM] Card 11 is silent on whether SkipGit skips the remote update

**Location:** `_lyx/plan/11-reset-moves-remote-branch.md` (skip-conditions sentence and `reset_test.go` tests).
**Issue:** The card, like D7, skips the update "under `SkipPush`" only.
The rest of fabric treats `SkipGit || SkipPush` as the push bypass (`PushWarpRebaseFreeAt`, `CoalescePushBothAt`), and `reset.go` will pass `EnvSyncOptions()`.
`internal/webstercli/reset_test.go`'s fixture sets `WEFT_SKIP_GIT=1` on every test, yet the card's CLI rows need the remote update to run under that fixture.
An implementer who follows the fabric convention makes those rows impossible; one who follows the card literally contradicts the convention without saying so.
**Fix:** State that `SkipGit` does not skip the fetch or the remote update (the reset itself is already a git operation it does not skip), and why, and keep the fixture's `WEFT_SKIP_GIT=1` for the new CLI rows.

### [MEDIUM] Card 4 drops sandbox scenario W2c without licence

**Location:** `_lyx/plan/04-drop-model-assertion.md` (Docs paragraph, SANDBOX-WEBSTER-SUITE sentence).
**Issue:** The card removes W2 whole "since the mechanism it armed is gone".
W2 holds three assertions; (a) and (b) test the model injection, but (c) checks that a fork's `subagents/<id>.jsonl` transcript is flushed by the time its report lands, which `record-batch`'s audit depends on and which D10 does not touch.
D10 licenses removing the model assertion, not that live check.
`internal/shuttleengine/claudeengine/modelswitch_test.go` also cites "the W2b corruption mode", which dangles once W2 is gone.
**Fix:** Remove only W2a and W2b (and the `assertedModel` tamper exception), keep W2c as a W1 assertion or its own scenario, and reword the dangling W2b reference.

### [MEDIUM] cli-drive rationale does not match what D8 requires

**Location:** `_lyx/plan/00-overview.md` (Shared Decision `cli-drive`).
**Issue:** The rationale says Publish cannot be driven through `lyx loom step` because "its resumed half reaches the GitHub step", which loomcli's harness cannot fake.
D8's acceptance item is only the push-rejected stuck reason, and that half halts before any GitHub call.
The resumed half is an extra the card adds.
The true obstacle is that no loomcli test steps the Publish row today, so a CLI drive needs a new loom-step harness.
**Fix:** Reword the rationale to the true reason, keep the departure from "driven through the `lyx` CLI" recorded, and say what verifies the D8 item instead (landingshed integration tier plus the loomcli closure test).

### [LOW] Card 11 leaves three tallies stale

**Location:** `_lyx/plan/11-reset-moves-remote-branch.md` (destructive-guard and docs paragraphs).
**Issue:** The card rewords `destroy.go`'s "nine executors" and the guard's floor comment.
`cmd/lyx/destructiveguard_test.go` also pins "nine banned tokens" and "destroy.go's nine executors" in its header and in the comment above `TestNoDestructiveBypass_FabricengineProductionSource`, and `internal/fabricengine/doc.go` pins "nine gate executors auto-record eight of the nineteen kinds".
Adding a tenth executor, a tenth token and a new Kind makes all of them false.
**Fix:** Name those comments and reword each so it counts nothing.

### [NIT] HEAD versus the hex-only contract

**Location:** `_lyx/plan/10-fabric-remote-only-commits.md` (`CommitsNotIn(tip, HEAD)`).
**Issue:** Card 9 returns `ErrInvalidSHA` for any argument that is not a hex object name, so a literal `HEAD` is refused.
**Fix:** Say card 10 passes the resolved HEAD SHA.
