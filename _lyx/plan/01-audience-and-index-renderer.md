# Card 1 — audience-and-index-renderer

**Create:**
- `plan:internal/clihelp#AudienceAnnotation` -> `const AudienceAnnotation = "lyx.audience"`
- `plan:internal/clihelp#AudienceOperator` -> `const AudienceOperator = "operator"`
- `plan:internal/clihelp#AudienceRole` -> `const AudienceRole = "role"`
- `plan:internal/clihelp#AudienceInternal` -> `const AudienceInternal = "internal"`
- `plan:internal/clihelp#Audiences` -> `var Audiences []string`
- `plan:internal/clihelp#IndexNoteAnnotation` -> `const IndexNoteAnnotation = "lyx.index-note"`
- `plan:internal/clihelp#Invocable` -> `func Invocable(cmd *cobra.Command) bool`
- `plan:internal/clihelp#RenderIndex` -> `func RenderIndex(root *cobra.Command, audience string) string`
- `internal/clihelp/index_test.go`

**Edit:**
- `internal/clihelp/annotations_test.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/clihelp#SkipStencilSeedAnnotation`
- `internal/clihelp#metaFlags`

**Intent:** Declare the `audience` annotation and build the command-index renderer in `internal/clihelp`, the package every `*cli` package and `cmd/lyx` already import, so the `help` module (card 2) and `orchcli` (card 14) call one renderer and no engine imports cobra (D1, D3).
The annotation key and its closed value set live in `annotations.go` beside `SkipStencilSeedAnnotation`; `Audiences` is the closed list, in the order operator, role, internal, and is the one place the set is spelled, so the `--audience` refusal and the tree test read it rather than restating it.
`IndexNoteAnnotation` names an optional annotation a group carries whose value the renderer appends, after one space, to that group's index line; card 9 puts the board payload legend there (D2, D7).
`Invocable` reports whether a command takes an `audience`: it is a leaf (no subcommands), or a runnable group, a command with subcommands and a `Run`/`RunE` of its own whose `Use` carries a token after its name or that has a local non-hidden flag of its own; `clihelp.GroupRunE` groups with no positional and no visible local flag are not invocable.
`RenderIndex` walks `root` in memory, reads no file and caches nothing, and prints the nested markdown list of D2: two spaces of indent per level, one `- ` line per command, `lyx` and the parent path implied by nesting.
A group line is the name in backticks, a colon and its `Short`; a runnable group renders in the leaf form and its children nest below it.
A leaf line is the name plus the rest of its `Use` in one backtick span, then, when the command has local non-hidden flags not already spelled as `--name` in `Use`, their `--name`s in one backticked run, then a colon and the `Short`; `--json`, `--verbose` and `--help` are never listed, and the flag set comes from `LocalFlags` minus hidden ones.
An invocable command prints only when its annotation equals `audience`; a non-invocable group prints only when a command under it prints; a command with no `audience` annotation never prints; the `completion` subtree is skipped by name; a command carrying `IndexNoteAnnotation` gets the note appended to its line.
`index_test.go` adds one test over a synthetic cobra tree (no `newRoot`, no process spawn, `t.Parallel`) that pins, as rows: the group form, the leaf form with its flag run, a flag already in `Use` omitted, the meta flags omitted, a runnable group in leaf form with its children nested, a group dropped when none of its leaves is in the audience, a hidden `internal` leaf shown only under `internal`, an unannotated leaf never shown, the `completion` skip, and the index note appended.
`TestAnnotationLiterals` gains rows pinning the five new literal values.
`cmd/lyx/testdata/test-budget.yaml` raises `internal/clihelp` by one for the new test.
Doc comments on every new symbol follow semantic line breaks.

**ImpactSummary:** New symbols only in clihelp plus two test files and one budget row; no existing behavior changes.
