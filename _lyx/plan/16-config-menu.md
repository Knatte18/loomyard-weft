# Card 16 — the module picker returns as lyx config menu

**Create:**
- `internal/configcli/menu.go`

**Edit:**
- `internal/configcli#Command`
- `internal/configcli#runConfig`
- `internal/configcli#buildConfigLong`
- `internal/configcli/configcli_test.go`
- `internal/configcli/doc.go`
- `docs/overview.md`

**Uses:**
- `cmd/lyx#TestCLITree_EveryCommand`
- `internal/configcli#editOne`

**Intent:**
Auto-assumption A1 removed the interactive picker with D9, and the operator overturned it: the picker returns as the `menu` subcommand, and bare `lyx config` keeps its cobra listing.

Restore `menu.go` from the base branch (`git show main:internal/configcli/menu.go`) and adapt it: `menu` drops its unused `*lyxcwd.Location` parameter and keeps the rest of its contract — print a numbered list of `configreg.Names()`, each marked `(configured)` or `(default)`, read one line from `in`, quit with exit 0 on `q`, refuse a non-number or an out-of-range number with exit 1, and otherwise route the chosen module to `editOne`.

Register a `menu` child on `Command`, beside `reconcile`, with `Args: cobra.NoArgs` and a `Short` that says it picks a module to edit interactively.
Its handler resolves the cwd and builds the real editor and sync exactly as `runConfig` does; extract that shared resolution out of `runConfig` into one helper both call, rather than copying it.
It reads from `cmd.InOrStdin()`.
The menu therefore resolves a cwd, which is unchanged from the picker's old behaviour; bare `lyx config` and the unknown-argument refusal still resolve none.

Bare `lyx config` lists `menu` among its subcommands through cobra's help, with no change to its `RunE`.
`TestNoModuleSharesNameWithSubcommand` already guards a module named `menu`.
`TestCLITree_EveryCommand` walks `config menu` as a leaf and requires its `Short`, so the tree walk covers it with no edit to `cmd/lyx`.

`buildConfigLong` says the interactive picker is `lyx config menu`.
`configcli`'s `doc.go` gains a paragraph for `lyx config menu`, saying it is the picker bare `lyx config` used to open.
`docs/overview.md`'s two `lyx config` lines (the task 008 status line and the config module line) name `lyx config menu`.

Tests, untagged, in `configcli_test.go`, restoring the base branch's `TestMenu*` tests against the new signature:

- selecting `1` edits and syncs the first module, and the listing names `board`.
- `q` exits 0 without calling the editor or sync.
- a non-number and an out-of-range number each exit 1 without calling the editor or sync.
- a seeded module is listed `(configured)` and an unseeded one `(default)`.
- bare `lyx config` from a non-git temp dir names `menu` in its output, alongside `reconcile` and every module, in `TestBareConfigListsModulesAndVerbs`.
- `lyx config menu bogus` from a non-git temp dir exits 1 with an error envelope and calls no editor.

Bound: `menu` adds one routing edge, reachable only by its exact name; it opens the editor only for a module in `configreg`, through the same `editOne` the module path uses.

**ImpactSummary:** the `config` command tree gains `menu`; bare `lyx config`, `lyx config <module>`, `--print`, `--set` and `reconcile` are unchanged.

**Commit:** 16: the module picker returns as lyx config menu
