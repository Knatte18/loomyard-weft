```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, find the test row that would fail if it broke.

**Per behaviour, the test that pins it**

- **Body without a `## Merge danger` section is refused** (`no-merge-danger`): `TestValidateDescription_Table` row "no merge danger".
- **Empty body gives only `empty-body`, with no merge-danger finding**: row "empty body".
- **Heading matched case-insensitively**: row "merge danger as list items in backticks" (`## merge danger`).
- **Door missing** and **door neither way**: rows "door missing" and "door neither way".
- **Door matched case-insensitively** (`One-way`): row "merge danger as list items in backticks".
- **Blast radius missing, two words or empty**: rows "blast radius missing", "blast radius two words" and "blast radius empty".
- **Section ends at a level-two heading**: row "labels under the next heading are outside the section".
- **Level-three heading stays inside the section**: row "a level-three heading stays inside the section".
- **Section ends at a level-one heading** (`isSectionHeading`, the `"# "` clause): **none**. This is mutation survivor `description.go:139`.
- **Labelled line as a `- ` list item, value in backticks**: row "merge danger as list items in backticks".
- **Labelled line as a `* ` list item**: none. Minor; folds into the finding below on `labelledValue`.
- **Trailing period stripped from the value** (`description.go:152`): **none**. The only row with a period is the blast radius `` `board`. ``, which stays one word with or without the strip. No door value ends in a period.
- **Labels on the line right under the heading** (`description.go:127` `end := i + 1`, `:131` `lines[i+1:end]`): **none**. Every row puts a blank line after the heading, so mutants that skip one line survive.
- **Draft ends with the placeholder Merge danger section**: `TestDraftBeamDescription`, every row (`wantMergeDanger` literal).
- **Unedited draft is refused when recorded** (the contract in `beamMergeDangerDraft`'s comment): **none**. `TestHubLandDeps_DraftThenRecordStampsAndCommits` replaces the placeholders before `RecordDescription`. No row records the draft as written and expects `merge-danger-door` and `merge-danger-blast-radius`.
- **Valid description with the section passes every caller gate**: `TestRecordBeamDescription` (valid const), `TestNewDescriptionGate` "valid", `TestGateParity_DescriptionGate` "Clean", `TestValidateDescriptionCmd` "Clean", `TestHubLandDeps_ReadyReads`, and the loomrecipe describe fixture.
- **Callers refuse a description without the section**: each caller delegates to `ValidateDescription`, pinned by the table above. No caller-level row, which is acceptable for a shared function.
- **`lym-glossary` shipped entry, aimed at `orch-prompt` alone and last in shipped order**: `TestStore/ShippedEntriesArePinned` row `{"lym-glossary","orch-prompt","## lym's glossary"}`.
- **Glossary reaches orch sessions only** (claude orch, numbered orch, codex orch; not writer or lookalike): `TestRender` rows ClaudeOrch, ClaudeNumberedOrch, CodexOrch, ClaudeWriter, ClaudeOrchLookalike and CodexWriter, plus the `Show(orch-prompt)` assertion.
- **Discussion prompt carries the filled glossary marker**: `TestProducerSpecs_SkillsAndParentDirective` renders the lead prompt through `seatengine.Prompts`. The stencil fill fails on a missing or empty required marker (`missingkey=error`, `internal/stencil/stencil.go:39`). Presence is pinned; content is not, and it has one source.
- **`withGlossary` panics on an invalid entry or a taken slug**: none. These are build-invariant panics on lym's own embedded data (programmer error), so no finding.
- **`shipped.go:58` survivor** (mutated constant `"\n"` in `TrimRight`): no test can kill it. `passagesOf` already trims `"\n"` (`render.go`, `strings.Trim(entry.Text, "\n")`), so no finding.

**Findings**

1. **LOW · design** · `internal/summaryparser/description_test.go` (`TestValidateDescription_Table`). Mutation survivors at `description.go:127` and `:131` are a missing check. No row puts the Door and Blast radius lines directly under `## Merge danger` with no blank line, so a section start shifted by one line goes unseen.
   - Evidence: `description.go:127,131`; every merge-danger row in `description_test.go` has `"## Merge danger\n\n"`.
   - Fix: add a row with `"## Merge danger\n**Door:** two-way\n**Blast radius:** docs\n"`, want nil.
2. **LOW · design** · `internal/summaryparser/description_test.go`. Survivor at `description.go:139`: no row ends the section at a level-one heading. The `"# "` clause of `isSectionHeading` can be deleted unseen.
   - Evidence: `description.go:139`; the table's only terminating heading is `## Notes`.
   - Fix: add a row with the labels after a `# Other` heading below the section, want `merge-danger-door` and `merge-danger-blast-radius`.
3. **LOW · design** · `internal/summaryparser/description_test.go`. Survivor at `description.go:152`: no row has a door value with a final period, such as `**Door:** two-way.`. The period strip that the doc comment at `description.go:143` promises is unchecked. The radius row `` `board`. `` passes without it.
   - Evidence: `description.go:143,152`; row "merge danger as list items in backticks".
   - Fix: make that row's door `` `One-way`. `` (or add a row), want nil.
4. **LOW · design** · `internal/landingshed/beamdescribe.go` (`beamMergeDangerDraft` comment: "a draft recorded unedited is refused"). No test records the unedited draft and expects refusal. The beamcli integration test edits the placeholders away first, so placeholders that happened to validate would pass every test.
   - Evidence: `beamdescribe.go` const comment; `deps_integration_test.go:236-244`.
   - Fix: add a `TestRecordBeamDescription` row recording the draft text unchanged, want the door and blast-radius findings.

Outside my lens: `render_test.go` builds its expected glossary passage as `strings.TrimRight(Glossary(), "\n")`, the same formula as `shipped.go:58`. The literal heading check in `store_test` keeps the glossary text itself pinned.

Tool calls:
1. grep for glossary assertions in loomengine, stencils and loomcli tests, and `TestDiscussionTable`'s loop. My context did not show whether any test pins the glossary marker.
2. grep of `TestProducerSpecs_SkillsAndParentDirective` and stencil.go's required-marker handling, to learn whether a missing or empty marker fails a test.
3. `discussion_test.go` lines 60-120, to confirm the test renders the Discussion prompt through `seatengine.Prompts`.
