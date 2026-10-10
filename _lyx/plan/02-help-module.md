# Card 2 — help-module

**Create:**
- `internal/helpcli/doc.go`
- `internal/helpcli/cli_test.go`
- `plan:internal/helpcli#Command` -> `func Command() *cobra.Command`
- `plan:internal/helpcli#RunCLI` -> `func RunCLI(out io.Writer, args []string) int`
- `plan:internal/helpcli#RunCLIIn` -> `func RunCLIIn(cwd string, out io.Writer, args []string) int`
- `plan:internal/helpcli#indexCmd` -> `func indexCmd() *cobra.Command`

**Edit:**
- `cmd/lyx#newRoot`
- `cmd/lyx#walkCommands`
- `cmd/lyx#visibleChildren`
- `cmd/lyx#TestCLITree_EveryCommand`
- `cmd/lyx#TestRegistration_AllModulesRegistered`
- `cmd/lyx#TestSandboxCoverage_AllModulesCoveredOrExcluded`
- `cmd/lyx#TestExitCode_HelpPaths`
- `cmd/lyx#TestJSONHelp_Schema`
- `internal/clihelp#renderCmdJSON`
- `internal/clihelp#TestInstallJSONHelp_RendersSchema`
- `tools/sandbox/SANDBOX-CORE-SUITE.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Prosa:**
- `docs/overview.md`

**Uses:**
- `plan:internal/clihelp#RenderIndex`
- `plan:internal/clihelp#Audiences`
- `plan:internal/clihelp#AudienceOperator`
- `plan:internal/clihelp#AudienceAnnotation`
- `internal/clihelp#SkipStencilSeedAnnotation`
- `internal/clihelp#AnnotationEnabled`

**Intent:** Add the `help` module (D1): `internal/helpcli` exposes `Command()`, `RunCLI` and `RunCLIIn` like every module, and its command replaces cobra's built-in help command on the root.
`Command()` returns a command with `Use: "help [<command>...]"`, a lower-case one-line `Short`, a `Long` under the cap, the `operator` audience, the stencil-seed skip annotation (it reads no stencils and stays silent), a `ValidArgsFunction` that completes command names from the root like the built-in does, and a `RunE` that resolves the topic word by word from `cmd.Root()`: bare `lyx help` prints the root's help; each word must name a child of the command reached so far, and the first word that names none is refused with a JSON error through `output.Err` whose text contains `unknown subcommand` and names that word, so `lyx help no-such-topic` and `lyx help board bogus` are both refused where the built-in's `Find` would print the board group's help for the second.
`indexCmd` is the `index` child: `Use: "index"`, `operator` audience, the seed-skip annotation, one `--audience` string flag defaulting to `operator`, whose value outside `Audiences` is refused with a JSON error naming the set, and a `RunE` that writes `clihelp.RenderIndex(cmd.Root(), audience)` to the command's output as text.
Both `RunE`s check `clihelp.ShouldAbort` first.
`newRoot` mounts it with `root.SetHelpCommand(helpcli.Command())` after `AddCommand`, then calls `root.InitDefaultHelpCmd()` so the command is in `root.Commands()` before execution, which is what the tree walks read; the "Available modules" sentence in the root's `Long` names `help`, since the walk's "root.Long names every mounted module" subtest now sees it.
`TestRegistration_AllModulesRegistered`'s oracle collects `SetHelpCommand` arguments of the `<ident>.Command()` form exactly as it collects `AddCommand`'s, so `helpcli` counts as registered.
`walkCommands` and `visibleChildren` stop skipping `help` and keep skipping `completion`; in `TestCLITree_EveryCommand` the bare-group check alone exempts the command named `help`, because bare `lyx help` prints the root's help rather than its own children, while the bogus check runs for it like any group.
`TestSandboxCoverage_AllModulesCoveredOrExcluded` stops skipping `help`, and the S0 Discovery scenario of `tools/sandbox/SANDBOX-CORE-SUITE.md` gains a `**Covers:** help` line and a sentence asking the agent to run `lyx help index`, which is how the module meets `PATTERN-sandbox-coverage`.
`renderCmdJSON` stops skipping `help` by name and keeps skipping `completion`, so `lyx --json` lists `help` with its `index` child; `TestInstallJSONHelp_RendersSchema` and `TestJSONHelp_Schema` assert `help` present and `completion` absent.
`TestExitCode_HelpPaths` gains rows for bare `lyx help` (names the modules) and `lyx help board` (names a board verb).
`cli_test.go` drives `RunCLI` and `cmd/lyx`'s `run` is out of reach from here, so it builds a small root with the help command mounted the way `newRoot` does and pins, as rows of one table test: `help` with a topic prints that command's help, `help index` exits 0 and prints the operator index of the fixture tree, `--audience role` and `--audience internal` print that audience alone, an unknown audience is refused naming the set, `help no-such-topic` and `help <group> bogus` are refused with `unknown subcommand` naming the word, and `__complete help ""` lists the command names.
`doc.go` holds the module doc and the index's design: what the index is for, the line form, the audience rule, that the renderer lives in `clihelp` so `orchcli` can call it, and that `lyx help [<command>...]` keeps cobra's behaviour plus `index`.
`docs/overview.md` adds `help` to the Modules list with its `index` verb and mentions `SetHelpCommand` in the Module dispatch paragraph.
`cmd/lyx/testdata/test-budget.yaml` gains an `internal/helpcli` row sized for the test file.

**ImpactSummary:** Root gains a module; two tree tests, the registration oracle, the sandbox guard, the JSON help renderer and its two tests change expectations; no module behavior changes.
