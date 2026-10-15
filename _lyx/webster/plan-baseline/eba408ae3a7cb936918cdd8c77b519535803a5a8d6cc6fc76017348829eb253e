# Card 5 — loom-wiring

**Create:**
- `internal/loomcli#TestWire_LanguageSkills`
- `internal/loomcli#loomCLI.wireLanding`
- `internal/loomcli#TestWireLanding_TellsLanguageSkills`

**Edit:**
- `internal/loomcli#loomCLI`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#loomCLI.wireDarn`
- `internal/loomcli#loomCLI.loomPreRun`
- `internal/loomcli#loomCLI.buildLoomShed`
- `contracts/specs/refusal-spec.md`

**Uses:**
- `internal/languageconfig#Load`
- `internal/darnengine#DarnInputs`
- `internal/websterengine#RunDeps`
- `internal/landingshed#Deps`

**Intent:**
`lyx loom`'s wiring loads the `language` module once per process and tells the list to the darn writer, the Webster master and the merge conflict session at Merge-In and at Finalize (D4).

`loomCLI` gains the field `languageSkills []string`, the hub's language skills `wire` loaded.
`wire` calls `languageconfig.Load(fabricengine.ConfigDirs(location))` right after it loads `landing.yaml`, returns its error unchanged (the error already names `skills` and its way forward), stores the list in `c.languageSkills` before `wireDarn` runs, and sets `runDeps.LanguageSkills` to it.
`wireDarn`'s `DarnTable` closure tells `LanguageSkills: c.languageSkills` in the `darnengine.DarnInputs` it builds.

`loomPreRun` and `buildLoomShed` each hold the same block from `fabricengine.Open(c.location)` through the `c.env.Landing = landingDeps(...)` assignment.
That block moves into the new method `wireLanding`, which opens the fabric, reads the landing branches and origin URL (an unusable origin URL still passes through as empty), builds the push closure from `fabricengine.EnvSyncOptions()`, and assigns `c.env.Landing = landingDeps(...)`, passing `c.languageSkills` where card 4 left `nil`;
`loomPreRun` and `buildLoomShed` call it in place of the block and return its error, so both production paths tell the list through one tested site.

What the new load can refuse: a loom verb whose wiring finds the hub's `language.yaml` present but unusable now halts before any row builds.
It is bounded by D3: an absent hub file resolves the empty list and never refuses.
`contracts/specs/refusal-spec.md`'s shed section gains a row after "hub config missing", its way forward cell quoting the loader's clause as the message carries it: `language.yaml invalid` | a loom verb's wiring finds the hub-wide `language.yaml` present with content that does not parse, a `skills` value that is not a list of strings, or an entry that is empty or carries leading or trailing whitespace, or a worktree local override file that holds `skills`, which the Guard classification refuses | correctness halt | fix it with "lyx config language" from the prime.
An unreadable `language.yaml` is not that row's trigger: it surfaces as the arming error that the shed section's `step arming refused` row already covers.

`TestWire_LanguageSkills` (in `internal/loomcli/wiring_test.go`, over `hubLocation`, writing `<BoardDir>/_lyx/config/language.yaml` with `configengine.ConfigFile(fabricengine.BoardDir(loc.HubPath), "language")`) is table-driven:
- `skills` listing `scribe:golang-comments`, `scribe:golang-testing` on a darn seed (`seedDarnHubConfig` as `TestWire_DarnRecipe` uses it, `recipe: shedrun.RecipeDarn`): `c.languageSkills` and `c.runDeps.LanguageSkills` equal that list, and the chair of the table `c.env.DarnTable(loomshed.DarnTold{})` returns loads `scribe:prose`, `scribe:code-quality`, `scribe:testing`, `scribe:golang-comments`, `scribe:golang-testing`;
- `skills` holding an empty entry: `wire` fails with an error containing `skills` and `lyx config language`, which reaches the new refusal row.
The absent-file degrade is card 1's `TestLoad_ResolvesSkills` row, and the existing hub wiring tests already run `wire` with no `language.yaml`.

`TestWireLanding_TellsLanguageSkills` (in `internal/loomcli/wiring_publish_integration_test.go`, `integration`-tagged, package `loomcli`) builds a real hub and pair as `TestWire_Real_PublishRejectedPushNamesRemoteTip` does, writes the hub's `language.yaml` listing `scribe:golang-testing`, runs `wire` and then `wireLanding`, and asserts `c.env.Landing.LanguageSkills` equals that list, so a `wireLanding` that told `nil` fails it.

**ImpactSummary:** `wire` gains one hub-file load, the duplicated landing block of `loomPreRun` and `buildLoomShed` becomes one method, and that method tells `landingDeps` the loaded list in place of card 4's `nil`; no other package calls any of them.
