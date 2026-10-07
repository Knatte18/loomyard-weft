---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: _lyx/plan/11-reset-moves-remote-branch.md:34
    summary: Card 11 makes the remote update the one push-shaped fabric step that ignores SkipGit, with a rationale that does not hold and no doc line recording the exception
  - id: F2
    severity: LOW
    class: scope
    location: _lyx/plan/04-drop-model-assertion.md:3
    summary: tools/sandbox/suite.go still describes the webster suite as exercising model-escalation scenarios after W2 goes, and is not an Edit target of card 4
  - id: F3
    severity: NIT
    class: consistency
    location: _lyx/plan/00-overview.md:48
    summary: The cli-drive decision lists card 12 under Applies to although its own text says card 12 is not driven through the lyx cobra command
---

### [LOW] Card 11's SkipGit exception rests on a false premise and is recorded nowhere durable

**Location:** `_lyx/plan/11-reset-moves-remote-branch.md:34-35`
**Issue:** The card states that `SkipGit` does not skip the fetch or the remote update, so it answers the question the focus names.
Its reason is that "the reset itself is a git operation `SkipGit` never skipped".
`ResetPairWarp` takes no `SyncOptions` today (`internal/fabricengine/destroy.go:1556`), so `SkipGit` has never reached it and the sentence is vacuous.
Every other push-shaped fabric step bypasses on `SkipGit || SkipPush`: `spawn.go:97` and `:129`, `pushanchored.go:48`, `weftwiring.go:140`, `add.go:162`, and `doc.go:493` for `Add`'s analogous remote delete ("`SkipPush`/`SkipGit` skip every probe and the replacement").
The decision record's D7 names `SkipPush` alone, so the card's choice is licensed and the CLI rows under `WEFT_SKIP_GIT=1` do run the update as the card says.
What is missing is that the one force push in fabric ignoring `SkipGit` is an exception to the convention, and the card's Docs list does not make `fabricengine/doc.go` say so.
**Fix:** Replace the rationale with the true one: D7 names `SkipPush` as the only bypass, and the CLI rows run under `WEFT_SKIP_GIT=1` because they exercise the update.
Add to the Docs list that `fabricengine/doc.go`'s `ResetPairWarp` paragraph states that `SkipGit` does not gate the remote update, unlike the other push-shaped steps.

### [LOW] Sandbox suite's Go description names a scenario card 4 removes

**Location:** `_lyx/plan/04-drop-model-assertion.md:3-21` (Edit list); `tools/sandbox/suite.go:129`
**Issue:** `suite.go` describes `websterSuite` as "exercising the lyx webster fork-loop and model-escalation black-box agent scenarios".
`SANDBOX-WEBSTER-SUITE.md` has no escalation scenario other than W2's model assertion, and card 4 removes W2a and W2b.
The card's grep list (`AssertedModel`, `assertedModel`, `Injector`, "model switch", "injected") does not match "model-escalation", and the file is not an Edit target, so the comment goes stale unseen.
**Fix:** Add `tools/sandbox/suite.go` to card 4's Edit list and reword the comment to name the fork-loop scenarios only.

### [NIT] cli-drive's Applies to contradicts its own Decision

**Location:** `_lyx/plan/00-overview.md:43-48`
**Issue:** The Decision says each fix's regression drives the cobra command, and then says card 12 is driven below the CLI.
Its `Applies to: cards 1–8, 11, 12` lists card 12 as following the rule it departs from.
The reason given for the departure holds: no `internal/loomcli` test steps the Publish row (the only `NamePublish` uses there are in approve and goto tests), and the plan names what verifies the acceptance item, `landingshed`'s integration tier plus `landingdeps_integration_test.go`.
**Fix:** Change `Applies to` to cards 1–8 and 11, and let the Decision's Rationale carry card 12's departure.

## Focus settled

- `initState` (card 4): its callers are `verbs_test.go`, `acceptaudit_test.go` and `restoreplan_test.go`, all Edit targets, and no other test file calls it.
  The `injector` seam, `AssertedModel` and `BeginDeps.Roles`/`Engine` have no other reader in `internal/websterengine` or `internal/webstercli`, so `go vet` compiles when the card is followed.
  Not a defect.
- W2 removal (card 4): W2a and W2b assert the injection and its idempotency, which D10 removes, and W2c (fork transcript flush) is kept and moved into W1.
  The `modelswitch_test.go` comment is an Edit target and `internal/websterengine/doc.go`'s "tamper-armed W2 scenario" sits in the section the card removes.
  The one stale site is F2.
- Master stencil (card 7): the lines that send a failed batch to `recover-batch` are the `stuck` and `failed` progress rungs, the `stuck` and no-report rungs, the `record-batch` `batch_failed` rung and the audit-findings sentence.
  The card orders a whole-stencil search, and the clauses `claims_test.go` pins sit on lines it keeps verbatim.
  Not a defect.
- New `recover-batch` refusal (card 7): the card picks the existing `batch_failed` flag, `internal/webstercli/recoverbatch.go` is an Edit target and the `recover-batch` `batch_failed` rung already ends the run stuck, so a rung exists.
  Not a defect.
- `RunResult.Warnings` (card 8): `internal/websterengine/doc.go` is an Edit target, and `summary.go`'s done-only wording stays as the card says.
  No other doc describes the old scope or text.
  Not a defect.
- Executor, token and kind counts (card 11): the sites are `destructiveguard_test.go` lines 10, 43, 145-146 and 193, `destroy.go:54` and `fabricengine/doc.go:566`, and the card rewords all of them.
  Only `livestate_mutationoracle_test.go` tabulates the kinds, and it is an Edit target.
  Not a defect.
- Publish drive (overview): see F3.
