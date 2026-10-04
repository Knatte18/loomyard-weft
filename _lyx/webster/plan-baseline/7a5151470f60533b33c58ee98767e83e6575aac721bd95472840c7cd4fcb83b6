# Card 5 — board-label-filter

**Edit:**
- `internal/boardengine/store.go`
- `internal/boardengine/board.go`
- `internal/boardengine/store_test.go`
- `internal/boardengine/board_test.go`
- `internal/boardcli/cli.go`
- `internal/boardcli/verbs_test.go`

**Uses:**
- `internal/boardengine/config.go`

**Intent:**
D6: `lyx board list` and `lyx board find` take a repeatable `--label <name>` flag, and an entry matches when it carries every named label (AND).

- `Store.ListTasksBrief` and `Store.Find` take the label filter and keep only matching entries, in the same README order.
- `Board.ListTasksBrief` and `Board.Find` take the filter.
  A config-built `Board` (one whose outputs are configured, the same predicate the write guard in `boardCriticalSection` uses) refuses a filter label that is in neither list of its `Vocabulary` and that no entry of any kind or status carries, naming the label and `board.yaml`.
  A label removed from config but still carried by an entry is therefore accepted, so such entries stay listable for relabelling.
  A path-only `Board` has no config to check and filters without validating the flag.
- `cli.go`: `--label` is a `StringArray` flag on `list` and `find`; their help names the flag, the AND semantics and the refusal.
  Update every other caller of the two `Board` methods, found by grep, to pass no filter.

Tests: `store_test.go` filters by one label and by two (only entries carrying both); `board_test.go` refuses an unknown filter label on a config-built `Board`, accepts a label removed from config that an entry still carries and returns that entry, and filters without refusal on a path-only `Board`;
`verbs_test.go` drives `list --label a --label b` and `find --label a <text>` end to end, plus the refusal.

**ImpactSummary:** Adds a filter parameter to two `Board` read methods and a flag to two verbs; callers without a filter see the same output.
