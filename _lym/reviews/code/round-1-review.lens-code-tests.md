```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, find the test row that would fail if it broke.

**Per behaviour, the test that pins it:**

1. **A part's `labels` key decodes into `Part.Labels`.**
   `TestParse/ValidProposal` (`internal/raddleparser/proposal_test.go:28`, `:44`) round-trips `validProposal()`, whose parts now carry Labels, through `json.Marshal` and `Parse`.
   Dropping the field or its tag fails it.
   Because the JSON comes from the struct itself, renaming the tag does not fail it; the existing keys are left unpinned the same way, so this is no finding.
2. **`labels-missing` fires on a part with no label.**
   `TestCheck/PartWithNoModuleLabel` (`check_test.go:102`).
3. **`label-unknown` fires on a label outside the vocabulary.**
   `TestCheck/PartLabelOutsideTheBoardsVocabulary` (`check_test.go:103`).
   The row also pins one finding per bad label: `mod-a` beside `ghost` yields exactly one `label-unknown`.
4. **A type label used as a module label is refused.**
   `TestCheck/PartLabelThatIsATypeLabel` (`check_test.go:104`).
5. **A part label the Sketch does not carry passes.**
   `TestCheck/ValidProposalPasses` (`check_test.go:33`).
   Part-b's `mod-b` is not among the Sketch's labels (`fixture_test.go:54`, `:72`).
   `TestParse/ValidProposal` also exercises it.
6. **The check runs over every part.**
   The merge helper now carries Labels (`check_test.go:209`), so the one-part shape rows stay valid.
   The rows above mutate only `Parts[0]`; a loop that read only the first part would still pass them. This is weak, but `ValidProposalPasses` plus a mutation of `Parts[1]` is not needed to pin the per-part branch, so no finding.
7. **`raddleBoardContext` fills `LabelVocabulary` from the board config's labels, not its types.**
   `TestRaddleBoardContext` (`raddlecontext_integration_test.go:40`, `:103-105`).
8. **Raddle-File files each part with its type and its own labels, never the Sketch's set.**
   `TestRaddleFile` (`raddlefile_test.go:314-322`).
   The run entry now carries the Sketch-only label `mod-sketch` (`:98`), so reverting to `entry.ModuleLabels` fails the test.
   The tapestry shape's merged part is pinned too (`:42`, `:316-318`).
9. **The project's labels are its type plus its parts' labels, each once, in first-named order.**
   `TestRaddleFile`'s project assertion (`raddlefile_test.go:310`) wants `[feature mod-a mod-b]` from parts `[mod-a]` and `[mod-a mod-b]`.
   The repeated `mod-a` pins the dedup and the order.
10. **The fixtures that feed the new check** (`raddleverbs_test.go:40`, `:62-63`; `raddlecheck_test.go:55`, `:61`, `:81-82`).
    These edits keep the existing tests valid and pin no new behaviour of their own.
11. **The spec, raddle stencil and rubric prose.** Static prose, exempt.

**Findings:**

- **LOW · consistency** · `internal/raddleparser/check.go:431-438`
  - **Summary:** No test asserts the Detail text of either new finding.
    No assertion checks that it names its way forward: the "list in labels the module label…" clause for `labels-missing`, and `lym board labels` or the operator's board-config edit for `label-unknown`.
    `TestCheck` compares only check names (`check_test.go:183-189`).
    The package already pins an analogous way-forward Detail in `TestCheck_DependsOnNoteDetailNamesTheWayForward` (`check_test.go:179-198`).
    The way-forward text these findings add could be dropped or broken with every test still passing.
  - **Fix:** extend `TestCheck`'s rows, or that detail test, into a table over the new check names, asserting that each Detail carries its way-forward clause.

Outside this lens, noted only: a part listing one label twice, such as `["mod-a","mod-a"]`, passes the check. It then fails at Raddle-File's board write, where `validateLabels` refuses the repeat (`internal/boardengine/labels.go:28-30`). That belongs to the correctness lens.

Tool calls:
1. Listed the `raddleparser` test files and grepped `proposal_test.go` and the loomcli/loomshed raddle tests for labels coverage and the valid baseline row. My context lacked `proposal_test.go`.
2. Read `internal/raddleparser/proposal_test.go:1-96` to see whether `TestParse` exercises the new `labels` key. My context lacked that file.
