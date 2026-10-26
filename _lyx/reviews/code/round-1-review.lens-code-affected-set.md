```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. For each change, this is the set I found, and whether the diff leaves a member wrong.

1. `RenderMasterPrompt` signature (new `parentDecisions` parameter). Callers are `internal/websterengine/runlevel.go:745` and `internal/websterengine/template_test.go:366`. The grep found no others. Covered.
2. The new required marker `parent_decisions`. The set is the stencil header comment, the `render.go` map, the required-marker row at `template_test.go:480`, and the fixture at `template_test.go:248`. The webster stencil is also referenced from `internal/webstercli/cli.go`, `internal/webstercli/cli_test.go`, `internal/websterengine/merriambase.go`, `contracts/stencils/stencils.go` and `tools/tokencount/calibrate.go`. No marker list or fill of the Master stencil turned up in them. I did not open them. Covered, with that caveat.
3. The stencil rule that a stop is never a question to the parent. Messages to the parent in the Master stencil are only at lines 120, 126 and 221, and those are the `waiting` rung and the decision rung. No stuck rung still names a message. The `internal/websterengine/doc.go` text (lines 245, 461 and 465) agrees. The orch stencil (`contracts/stencils/orch/orch-template-role.md:54`) says the driver hands a stop to the arbiter first, so it needs no edit. Covered.
4. The new rule-table row and `JudgesFirst`. The set is `decideDrive`, `facts`, the `driveActionArbiter` handling in the `drive.go:285` loop, `arbitrate` (`drive.go:512`), `internal/sheddriver/doc.go`, the `drivewiring.go` wiring, and the `driverules_test.go` and `drive_test.go` rows. `arbitrate` hands back on an arbiter refusal or hand-on, as the record states, so the eligibility bound holds. The batten driver is untouched, so its `Arbiter` seam leaves `JudgesFirst` nil. Covered, except finding F1.
5. `internal/discussionparser`: the `addedDecisionSpans` extraction. Its two users are `dropAddedDecisionEntries` and `AddedDecisionTexts`. Behaviour of the first is unchanged, apart from the `head` return value. Covered.

**F1 — MEDIUM, class consistency**
- Location: `PATTERN.md:34`, the `PATTERN-shed-stop-chain` entry. Evidence is the entry text and the new row at `internal/sheddriver/driverules.go:234`.
- Summary: the entry states "rule table, tackler, arbiter, then parent, for every recipe". The diff makes a blocked Webster stop skip the tackler, so the entry is now false for that row. The diff does not touch `PATTERN.md`.
- Basis: CLAUDE.md says a cross-cutting invariant goes into PATTERN in the same commit as the code. The decision record's constraint reads "the order for every stop the new row does not mark", which already admits the exception.
- Fix: amend the entry to add the `JudgesFirst` exception. It could read "a stop at a row the driver's `JudgesFirst` marks, Webster's, goes rule table, arbiter, then parent".

Nothing else in the affected set was left wrong.

Tool calls (4 of 6):
1. Grep of `PATTERN.md`, `pattern/`, `docs/` and `internal/` for the stop-chain wording, the `driveActionArbiter` sites, files referencing `webster-template-master`, and the `EscalationEligible` definition. I needed the docs and the other users of the stencil, and my context did not have them.
2. Grep of the Master stencil and `internal/websterengine/doc.go` for other "parent" message sites. My context had only the diff hunks.
3. Read of `internal/sheddriver/drive.go` lines 270 to 330, to see how the loop handles `driveActionArbiter`.
4. An awk read of `arbitrate`, which returned nothing because I guessed the wrong file. I then read `drive.go` lines 500 to 570 with Read, to check the eligibility and hand-back paths. That Read is a fifth call.
