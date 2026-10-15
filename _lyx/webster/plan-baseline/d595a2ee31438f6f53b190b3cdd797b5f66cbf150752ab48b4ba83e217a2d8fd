# Card 1 — language-config-module

**Create:**
- `internal/languageconfig/doc.go`
- `internal/languageconfig/template.yaml`
- `internal/languageconfig/template.go`
- `internal/languageconfig/config.go`
- `internal/languageconfig/config_test.go`
- `internal/languageconfig/testmain_test.go`
- `plan:internal/languageconfig#ConfigTemplate` -> `func ConfigTemplate() string`
- `plan:internal/languageconfig#ConfigSpec` -> `func ConfigSpec() configengine.Spec`
- `plan:internal/languageconfig#ErrUnusable` -> `var ErrUnusable = errors.New("language config unusable")`
- `plan:internal/languageconfig#Load` -> `func Load(dirs configengine.Dirs) ([]string, error)`
- `plan:internal/languageconfig#AppendSkills` -> `func AppendSkills(own, language []string) []string`

**Edit:**
- `internal/configreg#Modules`
- `internal/configreg#TestModules_SeedOnlyAndHubWideFlags`
- `internal/configreg#TestModules_LocalLayerRefusals`
- `pattern/PATTERN-config-strictness.md`
- `docs/overview.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/configengine#LoadOrTemplate`
- `internal/configengine#MarkInvalid`
- `internal/configengine#CategoryGuard`

**Intent:**
Add the hub-wide config module `language` (D1, D2, D3, D7) as the new package `internal/languageconfig`, modelled on `internal/loggerconfig`'s embed-and-accessor layout, and the one shared skill-merge helper every code role uses (D4).

`template.yaml` holds one key, `skills: []`, with comments in the template's own inline style saying: the skills every code-writing or code-reviewing role (the darn writer, the Webster master and recovery strand, the merge conflict session, the Burler seats of a code round) loads after its own, in load order;
comment and testing skills such as `scribe:golang-comments` and `scribe:golang-testing` are the expected content;
a build skill such as `scribe:golang-build` or `scribe:csharp-build`, whose raw commands bypass the hub's own build and test route, belongs only in a hub that has no such route.
The `skills` key is a hatch (D7): a listed build skill sends every code role to its raw commands, such as `goimports -w`, `golangci-lint run` or a one-package `go test`, which run unrefused outside the gate slots; bounding that are the template comment, the operator's per-hub choice, the Guard classification that keeps a pair from setting it, and the verify gates, which still run the hub's own build.
`template.go` embeds it behind `ConfigTemplate`.

`ConfigSpec` returns `Name: "language"`, the template, `HubWide: true` and `Categories: {"skills": configengine.CategoryGuard}`, so `LocalKeys` leaves `skills` out and a local override holding it is refused (D2).

`Load(dirs)`:
- When `dirs.Board` is empty it returns an empty list and a nil error and reads no file at all, so a standalone run never reads a worktree `_lyx/config/language.yaml` (D3).
- Otherwise it resolves through `configengine.LoadOrTemplate(dirs, ConfigSpec())`, so an absent hub `language.yaml` resolves the template's empty list.
- It decodes `skills` as a YAML list of strings and returns its entries in file order, repeats kept (`AppendSkills` drops them).
- Every content failure, a `LoadOrTemplate` error for which `errors.Is(err, configengine.ErrInvalid)` holds (an unparseable hub file, a local override holding `skills`), a `skills` value that is not a list of strings, an empty entry or an entry carrying leading or trailing whitespace, returns an error marked through `configengine.MarkInvalid` that also wraps `ErrUnusable`, names the key `skills` and ends with the way forward clause `; way forward: fix it with "lyx config language" from the prime`, the shape `batcher`'s retired-key error has.
  `ErrUnusable` marks a content failure whose error already carries its own way forward, so a caller that adds its own (card 6) can tell it apart.
- Any other `LoadOrTemplate` error, such as a present but unreadable `language.yaml`, is returned unwrapped and unmarked, so webster's `configLoadError` default case classifies it transient, and loom's `wire` returns it as the plain arming error, which names the unreadable file and carries no `lyx config language` clause.
- It never checks that a named skill exists.

`AppendSkills(own, language)` returns a fresh slice: `own` in order, then each `language` entry not already present, in order, so a skill repeated in `language` or already in `own` appears once, at its first position (D4).
It never writes into `own`'s backing array, since callers pass package-level role lists.

`doc.go` documents the package: the module, its one Guard key, the degrade (absent hub file or no board dir yields the empty list, and the empty list leaves every role's skills exactly as without the module), the way forward on a failure, and `AppendSkills`'s ordering rule.

`config_test.go` (package `languageconfig_test`, seeded with `os.MkdirAll`/`os.WriteFile` under `t.TempDir()` like `internal/loggerconfig/config_test.go`) adds three table-driven tests:
- `TestLoad_ResolvesSkills`, rows: no board dir with a worktree `_lyx/config/language.yaml` listing a skill resolves the empty list (the worktree file is not read); a board dir with no `_lyx` resolves the empty list; a board file holding the template resolves the empty list; a board file listing `scribe:a, scribe:b, scribe:a` resolves exactly that list in file order.
- `TestLoad_InvalidFailsNamingKey`, rows over a board file: an empty entry `""`, an entry `" scribe:a"`, an entry `"scribe:a "`, a scalar `skills: scribe:a`, unparseable YAML `skills: [`, and a valid board file beside a local override holding `skills`, whose row creates the anchor's `_lyx/` directory (`configengine.ConfigDir(anchor)`) as well as the local file (`configengine.LocalFile(anchor, "language")`) and passes both `Board` and `Anchor` in the `Dirs`, so the local layer stacks and its refusal of `skills` is reached; each asserts the error is both `configengine.ErrInvalid` and `languageconfig.ErrUnusable` and contains `skills` and `lyx config language`.
  A further row puts a directory where the board's `language.yaml` belongs and asserts the error is neither `configengine.ErrInvalid` nor `languageconfig.ErrUnusable`.
- `TestAppendSkills`, rows: an empty language list returns `own` unchanged; a language list is appended in its order; a skill repeated in the language list appears once at its first position; a language skill already in `own` is skipped; and appending to an `own` slice with spare capacity leaves the element past its length untouched.
`testmain_test.go` runs the package under `tmuxkit.Main`, as `internal/loggerconfig/testmain_test.go` does.

`configreg.Modules` registers `{Name: "language", Template: languageconfig.ConfigTemplate, HubWide: true, CheckLoad: checkLoad(languageconfig.Load), Spec: languageconfig.ConfigSpec}` between `landing` and `logger`, keeping the alphabetical order `TestNames` pins.
`TestModules_EveryTemplateKeyClassified` then covers the `skills` classification with no edit, since a hub-wide module absent from its `wantLocalKeys` map must have no local keys.
`TestModules_SeedOnlyAndHubWideFlags` adds `language` to the hub-wide set, in its condition and its doc comment.
`TestModules_LocalLayerRefusals` gains the row `{"language", "skills:\n  - scribe:a\n", "skills", false}`: writing `skills` to the local layer is refused.

`pattern/PATTERN-config-strictness.md` adds `languageconfig` to the degrading callers.
`docs/overview.md`'s package tree gains an `internal/languageconfig/` line after `internal/darnengine/`, naming the hub-wide `language.yaml` and the skills every code role loads after its own.
`cmd/lyx/testdata/test-budget.yaml` gains the row `internal/languageconfig: 3` in sort order.

**ImpactSummary:** one new package and one registry entry; the new module changes the registry fingerprint, so the next hub reconcile seeds `language.yaml` from its empty template, and no existing loader or caller changes.
