# Card 8 — darn-writer-record

**Edit:**
- `contracts/stencils/darn/darn-template-write.md`
- `internal/darnengine#DarnInputs`
- `internal/darnengine#DarnSpec`
- `internal/darnengine#newSpecInputs`
- `internal/darnengine#TestDarnSpec`
- `internal/darnengine#TestDarnSpec_Refusals`
- `internal/darnengine/doc.go`
- `internal/loomcli#loomCLI.wireDarn`
- `contracts/stencils#wordingClaims`

**Uses:**
- `internal/loomengine#DiscussionDecisionRecord`
- `internal/loomengine#DiscussionSupportLog`

**Intent:** Have the darn writer write the decision record and support log before any code, leave a valid record alone, and follow a parent's recorded correction (D1, D3, D6).

`DarnInputs` gains `DecisionRecordPath` and `SupportLogPath`, both required: `DarnSpec` refuses either empty by name, like its other required fields, and fills them as the `decision_record_path` and `support_log_path` markers.
`wireDarn` tells them `loomengine.DiscussionDecisionRecord(location)` and `loomengine.DiscussionSupportLog(location)`, loom's own paths (D1), so `darnengine` stays told its geometry.
The spawn's `OutputFiles` stay the change description alone.

`darn-template-write.md` gains a step between "This spawn's work" and "Make the change", renumbering the later steps: write the record before any code.
When `{{.decision_record_path}}` or `{{.support_log_path}}` is absent, or `lyx loom validate-discussion` reports findings, write both from the board entry and a coarse reading of the code, in the format `{{.specs_dir}}/loom/loom-discussion-spec.md` defines, which the stencil points at and restates none of (`PATTERN-producer-pointer-rule`).
The record carries the scope as the writer reads it and the calls it makes before coding; a call made later while coding goes into the change description's design decisions.
The support log's `## Interview` states that a darn run holds no interview, and its `## Rejected alternatives` and `## Question ledger` carry the writer's own considered options and self-picks.
Run `lyx loom validate-discussion` until it exits 0 before making any change; lyx commits the two files at that clean run, and the writer never commits them.
Once it passes, the writer never edits either file again, in this spawn or a later one; a later spawn that finds both present and passing leaves them as they are.
Step 1 adds that the record's `Added after Discussion` entries, once present, amend the board entry and win over its body where they differ.
A new section, "A scope correction from your parent", says the parent records a correction with `lyx loom decision add`; on such a message the writer re-reads the record's `Added after Discussion` entries and follows them over the board body; when the message names a correction the record does not yet hold, the writer asks the parent to record it, follows the message meanwhile, and names it as an unrecorded parent correction in the change description's design decisions.
The writer never runs `decision add` itself, since the verb refuses a task session.
"What happens after you hand off" adds that the handoff also checks the record and support log, re-prompting in the same session with the findings.
The header comment lists the two new required markers.

`newSpecInputs` sets both paths; `TestDarnSpec`'s first-spawn row wants both paths and `<specs>/loom/loom-discussion-spec.md` in the prompt; `TestDarnSpec_Refusals` gains an empty-field row for each.
The `darn-template-write.md` row of `wordingClaims` gains claims on `lyx loom validate-discussion`, `loom-discussion-spec.md`, `Added after Discussion`, `never edit`, `unrecorded parent correction`, `holds no interview`, the respawn rule's "leave them as they are" and its write condition (a file absent or `validate-discussion` reporting findings), and the writer never running `decision add`, and a mustNot claim on "`## Auto-mode assumptions`", so the stencil restates no format.
`internal/darnengine/doc.go` says the session writes and validates the decision record and support log before it changes code and follows the record's added entries over the board body.

**ImpactSummary:** the darn writer's prompt gains the record step and two required path markers; its only producer of inputs, `wireDarn`, fills them.

**Commit:** `8: the darn writer writes the decision record before any code`
