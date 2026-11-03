---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: internal/boardengine/rename.go:31
    origin: lens:code-correctness
    summary: Board.Rename drops the saved rename's result when only the decision-record step fails
    fix: In internal/boardengine/rename.go, Board.Rename returns RenameResult{} on any error from boardCriticalSection, although the rename is already saved when followDecisionRecords fails. Give the post-save record error its own type in internal/boardengine/decision.go, and in Board.Rename return the result together with that error (result.(RenameResult), err) instead of an empty RenameResult. In internal/boardcli/cli.go at the b.Rename call, print the error beside the result's mentions list when the error is that type. Update the Rename doc comment to say the result accompanies a record error.
  - id: F2
    severity: LOW
    class: consistency
    location: internal/loomcli/decision.go:1270
    origin: lens:code-affected-set
    summary: The decision add --from help lost its operator-facing contract to fit the Long cap, and the upsert help dropped the front and on_land descriptions
    fix: In the add command's Long text in internal/loomcli/decision.go, restore in shortened form that --by and --amends-plan apply to every entry, that each live agent gets one notice naming the headings, and that two entries may not share a title or a PATTERN entry, trimming other prose to stay under the cap. In internal/boardcli/cli.go, restore a one-line description of front and on_land in the upsert Long optional-fields list beside the new discussed mark, again within the cap.
  - id: F3
    severity: LOW
    class: consistency
    location: internal/boardcli/cli.go:767
    origin: handler
    summary: The missing-record message spells the record file name as a literal decision-%s.md instead of forming it through the one named place
    fix: In internal/boardengine/decision.go export a function (for example DecisionFileName) that wraps decisionFileName, and in internal/boardcli/cli.go line 767 build the file name with it, keeping the message text otherwise as is. PATTERN-named-constants wants the record prefix and suffix declared once and referenced everywhere.
  - id: F4
    severity: NIT
    class: scope
    location: internal/boardengine/decision.go:202
    origin: lens:code-correctness
    summary: The orphan removal at the new slug runs for an unflagged entry too, while the parent's added decision names a flagged entry
    fix: In followDecisionRecords in internal/boardengine/decision.go, either pass the renamed entry's Discussed flag in and run the removal only when it is true, or add a half-sentence to the function's doc comment saying the removal also runs for an unflagged entry because a file at a fresh slug's name is always an orphan. No behavior change is required.
---

### [LOW] Board.Rename drops the saved rename's result when only the decision-record step fails

**Location:** internal/boardengine/rename.go:31
**Issue:** `boardCriticalSection` returns `(result, recordErr)` with the write saved, but `Board.Rename` turns any error into `RenameResult{}`. The operator loses the `mentions` list that the verb exists to give, and the entry is already renamed. The decision requires only that the failure is reported as the rename's error after the save, so this is a quality gap and not a contradiction.

### [LOW] The decision add --from help lost its operator-facing contract to fit the Long cap, and the upsert help dropped the front and on_land descriptions

**Location:** internal/loomcli/decision.go:1270
**Issue:** The cap fix (d4633f36) moved the `--from` notice and uniqueness rules into `internal/loomcli/doc.go`, which an operator reading `--help` never sees. The upsert `Long` merged `front` and `on_land` into one line with `discussed` and dropped their description. Neither edit is asked for by the decision record.

### [LOW] The missing-record message spells the record file name as a literal decision-%s.md instead of forming it through the one named place

**Location:** internal/boardcli/cli.go:767
**Issue:** `boardengine` declares the prefix and suffix once and forms the name in `decisionFileName`; the CLI message writes the name again as a literal, so a change of the prefix leaves it stale without a signal.

### [NIT] The orphan removal at the new slug runs for an unflagged entry too, while the parent's added decision names a flagged entry

**Location:** internal/boardengine/decision.go:202
**Issue:** A rename whose old record file is absent removes any file at the new slug's name, flagged or not. The behavior is a harmless superset, since a file at a fresh slug's name is always an orphan, but it is wider than the amendment says.

## Rejected

- Orch-role and stencil text does not mention `lym loom decision store` or `record_store_error` (lens:code-affected-set, MEDIUM scope): the decision record's In list does not include the orch role, and the refusal and the `record_store_error` text themselves name `lym loom decision store`.
- No test for a darn run or a beam project record not being copied through `decisionVerb` (lens:code-tests, LOW): `PastDiscussionApproval` pins darn and pawl as not past in `internal/loomrecipe/discussionapproval_test.go`, and the `decisionVerb` rows cover the not-past path.
- No test that `ApproveDiscussion` runs on every approving path and before the commit seam (lens:code-tests, LOW): `approveAndCommit` is the single call site of the Approve seam for all four paths, and the existing approve-before-commit tests cover its order; the new seam adds a resolver row, which `TestBouncerEntry_ApproveSeam` pins.
- No test for a failed removal of a removed entry's record file (lens:code-tests, LOW): the failure is only logged by design and cannot be provoked portably.
- The design_prefix refusal's way-forward text is not asserted (lens:code-tests, LOW): `internal/boardengine/config_test.go:178` asserts `set design_prefix in board.yaml`.
