# Card 9 — board-payload-use

**Create:**
- `plan:internal/boardcli#payloadKeys` -> `type payloadKeys struct`
- `plan:internal/boardcli#payloadLegend` -> `const payloadLegend = "A payload is one JSON object as a single quoted argument; ? marks an optional key, a|b exactly one of the two, <task> stands for upsert's payload, and an unknown key is refused."`
- `plan:internal/boardengine#UpsertPayloadKeys` -> `func UpsertPayloadKeys() (required, optional []string)`
- `internal/boardcli/payload_test.go`

**Edit:**
- `internal/boardcli#Command`
- `internal/boardcli#storeVerbs`
- `internal/boardcli#resolveLookup` -> `func resolveLookup(raw []byte, keys payloadKeys) (any, map[string]any, error)`
- `internal/boardcli#intakeCommand`
- `internal/boardcli#decodeIntakePayload` -> `func decodeIntakePayload(payload string, keys payloadKeys) (map[string]json.RawMessage, error)`
- `internal/boardcli#intakeImport`
- `internal/boardcli#intakeClose`
- `internal/boardengine#upsertAllowedKeys`
- `internal/boardengine#Store.SetDeps` -> `func (s *Store) SetDeps(selector any, dependsOn []string) error`
- `internal/boardengine#Board.SetDeps` -> `func (b *Board) SetDeps(selector any, dependsOn []string) error`
- `internal/boardengine/board_test.go`
- `internal/boardengine/store_test.go`
- `internal/boardengine/runlock_test.go`
- `internal/boardcli/help_test.go`
- `internal/boardcli/cli_test.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/clihelp#AudienceAnnotation`
- `internal/clihelp#AudienceOperator`
- `internal/clihelp#AudienceInternal`
- `internal/clihelp#IndexNoteAnnotation`
- `internal/boardengine#Store.SetStatus`
- `internal/boardengine#Store.GetTask`

**Intent:** Bring `board` to the D6 to D8 form and state the payload grammar once (D7).
`payloadKeys` declares, per payload command, its required keys, its optional keys, its exclusive pairs (exactly one of two) and, for a nested object key, that key's own `payloadKeys`; each parser validates against one such value declared once beside its command: `resolveLookup` takes the addressing verbs' value (`slug|id` plus `status` for set-status and merge's `set_status`, plus `depends_on` for set-deps), `decodeIntakePayload` takes `import`'s and `close`'s, and `upsert-batch`, `merge` and `set-deps` validate their wrapper keys through the same type instead of their inline loops; the unknown-key refusal text stays as it is.
`boardengine.UpsertPayloadKeys` exports the task payload's declaration, required `slug` and the remaining keys of `upsertAllowedKeys` as optional, and `upsertAllowedKeys` is derived from that one declaration so the two cannot drift.
No board command says `[json-payload]`; each `Use` names its keys as one `{…}` token: `get {slug|id}`, `remove {slug|id}`, `promote {slug|id}`, `set-status {slug|id, status}`, `set-deps {slug|id, depends_on}`, `upsert {slug, title?, brief?, body?, depends_on?, isolated?, status?, kind?, labels?, recipe?, priority?, short_name?, issues?}`, `upsert-batch {tasks: [<task>]}`, `merge {remove_slugs?, upsert: <task>, set_status?: {slug|id, status}}`, `intake import {issue, slug|into, title?, brief?, labels?}`, `intake close {issue, reason, completed?}`; `issues?` joins `upsert`'s token because the parser accepts it, and D7's rule that `Use` equals the parser's declaration outranks its illustrative list (Shared Decision `upsert-issues-key`).
`payloadLegend` is the one statement of the grammar: it ends the `board` group's `Long`, and `Command` sets it as the group's `IndexNoteAnnotation` so the index's `board` line ends with it; each verb's `Long` lists its fields as today and no longer restates the grammar.
The D9 cap is checked by measurement: after these edits, the implementer measures every board page's `Long` (`lyx board <path> --help --json | jq -r .long | wc -c`, groups included) and trims every page over 2,000 bytes; when this plan was written none was over.
`set-deps` resolves its target through `resolveLookup` like `get`, so `id` works: `Board.SetDeps` and `Store.SetDeps` take the selector `any` (a slug string or a float64 id, as `SetStatus` does) and `Store.SetDeps` finds the task the way `Store.SetStatus` does; its key set is otherwise unchanged and still refuses unknown keys; the three boardengine test files calling `SetDeps` pass a slug selector.
Every board `Short` starts lower-case and says when to reach for the verb (all are capitalised today), including the group's `task-tracker board`; every leaf carries `operator` except `sync` and `rerender`, which lyx's detached sync child and the maintenance path run and which carry `internal`.
`payload_test.go` adds one table test that, for each payload command, parses its `Use` `{…}` token (a brace-balanced span split on commas outside braces and brackets, `?` marking optional, `a|b` an exclusive pair, `key: {…}` a nested object, `<task>` and `[<task>]` the upsert declaration) into required, optional and exclusive sets and compares each with the command's declared `payloadKeys`, nested objects included, and that fails when a marker on either side disagrees, which it proves with one synthetic mismatch row.
`TestHelpSchema_LeafCommands` drops its `[json-payload]` expectations if any and keeps asserting the field names and the slug-limit sentence; `cli_test.go`'s lookup-contract step gains `set-deps` by `id` succeeding and `{"slug":…,"id":…}` refused as for `get`.
`cmd/lyx/testdata/test-budget.yaml` raises `internal/boardcli` by one.

**ImpactSummary:** Two boardcli parsers re-signed with all callers in the same two files, `SetDeps` re-signed with its CLI caller and three test callers; payload acceptance is unchanged except `set-deps` gaining `id`.
