```yaml
lens: code-affected-set
outcome: reported
```

code-affected-set lens: I traced everything outside the touched lines that the diff's changes affect, and checked which parts of that set the diff left wrong.

**Per change: what it affects, and whether that is still right**

1. New field `driftengine.Deps.DecisionRecord`:
   - Production builders of `Deps`: `internal/loomcli/driftwiring.go:249` sets it from `tables.decisionRecordPath`, which `wiring.go:1252` fills from `loomengine.DiscussionDecisionRecord`. Covered.
   - `internal/driftcli/wiring.go:200` builds `Deps` without a `JudgeSnapshotDir`, so it never takes a judge snapshot. Unaffected.
   - Test builders (`internal/testkit/envkit/envkit.go:744`, `shedrecipe` fixtures) leave it empty. Per the field's own doc, empty exempts nothing. Unaffected.
   - Docs: the field's doc comment and `doc.go` both describe it. Covered.

2. The judge snapshot directory now also holds `decision-record.md`:
   - Readers are `TakeJudgeSnapshot` and `appendedDecisionRecord`, both in the diff.
   - The restore text in `verify.go:646` (`changedSince`) only says to keep the snapshot directory, which still holds.
   - `doc.go` §TakeJudgeSnapshot is updated. Covered.

3. Decision-record exemption in `JudgeSnapshot.Changes`:
   - Callers: `verify.go:589` and `verify.go:619`, through `changedSince`.
   - Docs: the `Changes` doc and `doc.go`'s exemption and bound paragraphs. Covered, except that the bound wording is too narrow (finding 2).
   - PATTERN-sole-parsers: `driftengine` reads the record only through `internal/discussionparser`, so it complies.
   - `discussionparser`'s leaf allowlist (`internal/discussionparser/leaf_enforcement_test.go:17`) limits only what that package imports, not who imports it. Unaffected.

4. New `ParseExpectation` refusal for a `lines` entry on a baseline case whose every line is keyed. Callers:
   - `expect.go:349`: the Drift-Expect gate hands the message back to the agent. Covered.
   - `amend.go:156`: `lym drift amend --entries` refuses such an entry. Covered.
   - `check.go:244` (`sealedAndAmended`, parsing the sealed expectation) and `check.go:252` (each amendment): not right; see finding 1.
   - Docs: the `ParseExpectation` doc and `doc.go` §expected.yaml are updated.
   - Drift-Expect format text: `seats.go` `linesExpectationFormat` is updated. The stencil `contracts/stencils/drift/drift-template-expect.md` carries no range wording of its own, so nothing there goes stale. Covered.

5. Commit messages: there are no `Pattern-Departure:` trailers. The claims in `f52e4a45` and `03fc2871` match the diff.

6. Declared PATTERN changes: the directive declares none, so nothing is owed.

**Findings**

1. **MEDIUM, design. `internal/driftengine/expectation.go:297`, reached through `internal/driftengine/check.go:244`.** A run whose `expected.yaml` was sealed before this change, with a range over a fully keyed case, now fails every drift check.
   - Evidence: `check.go:244-246` re-parses the sealed bytes with `ParseExpectation` and returns `drift: the sealed expectation does not parse` before any amendment is merged.
   - Effect: issue #141's own workaround, `lym drift amend --entries` with key entries, can no longer get such a run past the gate.
   - The refusal's advice, "name each keyed value by a keys entry", names no step that can change a sealed file.
   - Before this change those cells were reported as uncovered and the amend workaround cleared them.

2. **LOW, consistency. `internal/driftengine/doc.go:274-277`, and the bound in `_lym/landing/summary.md`.** The stated bound is narrower than what the code lets through.
   - The doc says only "an added entry or a carry-over block the judge session writes" goes unseen.
   - `discussionparser.IdentityText` (`internal/discussionparser/identity.go:11-23`) strips every well-formed carry-over block from both the copy and the current record.
   - `appendedDecisionRecord` (`judgesnapshot.go:331-332`) checks only that added entries form a subset; it never compares carry-over blocks.
   - So a judge's edit to an existing carry-over block, or its removal, also goes unseen. The doc and summary don't say so.

Tool calls (all three were Bash):
1. Read `check.go` 236–262 and `expect.go` 340–360, and searched for import-guard tests that name `discussionparser`: my context had only the `ParseExpectation` call sites, not how each one handles the error, nor any importer guard.
2. Searched stencils, specs and docs for range wording and judge-snapshot wording: my context held no stencil or spec text that describes these outputs.
3. Read the import guard in `discussionparser/leaf_enforcement_test.go`, and searched `contracts/` and top-level `docs/` for files that mention `expected.yaml`: needed to see whether the guard limits importers and which stencils describe `expected.yaml`.
4. Searched the Drift-Expect stencil `drift-template-expect.md` for range and key wording: needed to see whether its wording goes stale beside the updated format text.
