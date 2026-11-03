```yaml
lens: code-tests
outcome: reported
```

Lens code-tests: for each behaviour the diff adds or changes, the test that pins it, or none.

**Per behaviour**
- Assistant seat kind in `Table.Validate` (mixed kinds, missing directives, end marker missing, shared or on a non-assistant, required or writer assistant, assistant lead): pinned by the `table_test.go` rows (`an assistant table whose lead includes only the advisor chair directive`, `a table of assistants and an advisor`, `two assistants sharing an end marker` and the others).
- Missing engine block passed over for an assistant in `Validate`: pinned by the `table_test.go` row `...is passed over`. The chair-never-passed-over side is the unchanged pre-existing refusal, not new behaviour.
- Missing engine block passed over in `CheckMarkers`: none found. A grep for `CheckMarkers` next to assistant in the seatengine tests returned nothing. The `...is passed over` row looks like `Validate`'s, so I could not confirm it. See F1.
- End marker written once on every ending: `TestEngine_Run_AssistantEndMarkerNamesTheEnding` has rows for done with no gate outcome, done with gate passed, gate failed, died, timeout, stopped (still running when the chair ends), skip reason, start error and missing engine block. It has no row for the wait-error ending, `wait error: <err>`. See F2.
- Stale marker and outputs removed before the chair starts: `TestEngine_Run_AssistantRemovesStaleFilesBeforeTheChairStarts`.
- Hold deferral counts an unended assistant mid-turn (fresh and resumed, idle, ended done): the four `engine_test.go` rows (`an unended assistant mid-turn defers`, `an assistant reading idle does not defer`, `an assistant that ended done never defers`, `an attached chair defers on an assistant the probe found`).
- Held turn end silent and no up line for an assistant: no dedicated assertion seen. The diff adds the skip only as the `!seat.Assistant` guards in `Run` and `Resume`, and the `watch` early return. The end-marker table does not appear to assert that no notice reaches the chair. Not reported as a finding because I could not confirm it.
- Every assistant stopped and marked stopped when the chair ends: the `still running when the chair ends` row, and `TestEngine_Resume_Assistants` for `found, still running`.
- Resume (found, ends done, not found, keeps an existing marker): `TestEngine_Resume_Assistants`. The unreadable-marker branch of `markLostAssistant` has no row. That is a log-only path, so I did not raise it.
- Chair and assistant prompt rendering (no-parent directive, engine-owned edit directive, `assistants` marker): `TestPrompts_AssistantTable`, with the `CodexAssistantEditDirective` and `ClaudeAssistantEditDirective` rows.
- `editdirective.EngineDirective`: `TestEngineDirective_ReadsTheEnginesCopyWithoutBanner` and `TestEngineDirective_EngineWithoutACopyNamesTheStencil`.
- burlerengine assisted round (table, report and marker paths, composition, unstarted assistant, copy only when done, probe and resume, assist-rules byte-for-byte for a plain round): `TestAssistPaths`, `TestEngine_AssistSeat`, `TestEngine_Review_AssistedRound`, `TestEngine_Review_AssistantThatDoesNotStart`, `TestKeepAssistReports`, `TestEngine_ProbeReview_AssistantSeat`, `TestComposeReviewPrompt_AssistRules`. The existing `prompt_test.go` call sites now pass `""` for the new argument, which covers the unchanged bytes.
- Resumed-path copy at conclusion: the `assist_test.go` hits at lines 257 and 358 suggest coverage, but I did not read the bodies to confirm they go through `ResumeReview`.
- `ValidateAssistModelList` and the config keys: the `config_test.go` rows (per-round list with empty entries, fan keys, zero `review_timeout_min` loads, local layer, malformed entries, unknown fan, negative timeout), `TestResolveReview_Assist` and the `tapestry_test.go` plan rows. There is also a direct `TestValidateAssistModelList`. See F3.
- `CapAssistTimeout`: `TestCapAssistTimeout`. `Config.StartAllowance`: `TestConfig_StartAllowance`.
- Await cap from `KeepWarm` in `internal/loomcli/wiring.go`, the `agentname.Format` fallback and the role `burler-review`: none. See F4.
- Review row (`pick`, attempt recorded before each spawn, `<round>b`, probe from record, record without the field reads as bare round, copy in the archive set): `TestBurlerReviewProducer_Call_AssistPick`, `TestBurlerReviewProducer_Call_RecordsAttempt` (including archiving the copy on retry) and `TestBurlerReviewProducer_Call_ProbeFromAttemptRecord` (both record rows).
- Wiring of assist settings and the capped bound into each segment: `loomcli/wiring_test.go`, with default and set-key cases for all three segments.
- Stencil markers: the `markers_test.go` rows. Stencil prose is exempt.
- Llm-tier Codex smoke test: `TestSmokeBurlerReviewCodexAssistant`.

**Findings**

- F1, LOW, scope. `internal/seatengine/prompt.go`, `CheckMarkers` (the `errors.As(err, &missing)` branch that passes an assistant over). No test was found that drives `CheckMarkers` with an assistant whose engine lacks a block. If the branch were removed, a table the engine can start would be refused before any seat starts. The fix is to add a row or assertion to the existing `CheckMarkers` test, with no new test function. Unconfirmed: the `Validate` row may cover it.
- F2, LOW, scope. `internal/seatengine/assistant.go`, `endingOf`, the `waitErr != nil && !stopped` branch giving `wait error: <err>`. The ending table in `TestEngine_Run_AssistantEndMarkerNamesTheEnding` has no row for it. The summary and the decision record both list "the wait error" as an ending the marker must name. The fix is a `waitErr` row in that table, plus one for the stopped-with-wait-error case, which gives `stopped`.
- F3, LOW, consistency (test economy). `internal/burlerengine/modelkeys_test.go`, `TestValidateAssistModelList`, a per-helper test. The `config_test.go` malformed-entry rows (`malformed discussion_assist_reviewer_model entry` and `malformed later code_assist_reviewer_model entry`) and the tapestry plan rows already cover the same refusals at the public surface through `LoadConfig` and `LoadTapestryConfig`. The fix is to fold any assertion the direct test alone makes (the way-forward text, 1-based index) into those rows, then delete it.
- F4, LOW, scope. `internal/loomcli/wiring.go`, the `awaitCap` resolution. `wiring_test.go` only pins the default 60-minute bound, which fits under the default timeouts, so a wrong or ignored await cap would not fail any test. The fix is a wiring test row with a keep-warm cap and a `review_timeout_min` that make the cap bind, asserting the resulting `Timeout` or `SkipReason`.

**Tool calls**
1. `git diff ... -U0 -- '*_test.go'` with a grep for test names and rows: my context held the production diffs but not the added tests.
2. `git diff -U0` of the seatengine, loomcli, loomengine, burlerengine and shedadapters test files, filtered for case names: to see the rows the first listing did not show.
3. A grep for wait-error, held-turn, `CheckMarkers`, archive, `ResumeReview` and await-cap coverage, plus the added `wiring_test.go` lines: to confirm which of those behaviours have a test.
