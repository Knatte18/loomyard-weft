# Card 6 — lyx board labels

**Edit:**
- `internal/boardcli#Command`
- `internal/boardcli#storeVerbs`
- `internal/boardcli/verbs_test.go`
- `plugins/ly/skills/board/SKILL.md`
- `tools/sandbox/SANDBOX-CORE-SUITE.md`

**Create:**
- `internal/boardcli/doc.go`

**Uses:**
- `plan:internal/boardengine#Label`
- `internal/boardengine#LoadConfig`

**Intent:**
Add the `labels` verb (D4): no payload, prints `{"ok":true,"types":[{"label":…,"description":…}],"labels":[…]}`, each list in `board.yaml` order, with an empty list written as `[]`.
`Command` keeps the `Config` its `PersistentPreRunE` loads in a closure variable beside `b`, and the verb reads `Types` and `Labels` from it; it reads `board.yaml` only, through that existing load, and resolves no cwd of its own.
Under the hidden `--board-path` the config is path-only, so both lists print empty.
The verb has a `Short`, a `Long` with an example, JSON errors through `internal/output`, and checks `clihelp.ShouldAbort` first (`PATTERN-cli-cobra`).
`storeVerbs`' `list` help example drops `undecided` for a label the template still has.

Tests, integration-tagged with the package's other verb tests: `lyx board labels` on a map-shaped `board.yaml` prints both lists in file order with descriptions; on a list-shaped one it prints the names with empty descriptions.

`SKILL.md` (D12): read `lyx board labels` before choosing labels; a label names the lyx module an entry mainly touches, and an entry that belongs to no module carries none beyond its type label; `types` and `labels` are maps edited with `lyx config board` (`--set labels.<name>=…` or the editor); the `undecided` examples go.
`SANDBOX-CORE-SUITE.md`'s S3 scenario runs `lyx board labels` and watches for both lists in file order (`PATTERN-sandbox-coverage`).
`doc.go` gains the package doc for `boardcli`: the verb tree, payload conventions, `--body-file`, and the `labels` verb.

**ImpactSummary:** one new board verb, the board CLI's config capture, the board skill and the sandbox S3 scenario.

**Commit:** 6: lyx board labels
