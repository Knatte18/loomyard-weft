# Card 5 — mount-shortcuts

**Edit:**
- `cmd/lyx#newRoot`
- `cmd/lyx#TestCLITree_EveryCommand`
- `cmd/lyx#TestSandboxCoverage_AllModulesCoveredOrExcluded`
- `cmd/lyx/testdata/test-budget.yaml`
- `docs/overview.md`
- `pattern/PATTERN-cli-cobra.md`

**Create:**
- `cmd/lyx/shortcuts_test.go`
- `plan:cmd/lyx#shortcutProblems` -> `func shortcutProblems(root *cobra.Command, shortcuts []shortcutcli.Shortcut) []string`
- `plan:cmd/lyx#TestShortcuts_FileAndRootListing` -> `func TestShortcuts_FileAndRootListing(t *testing.T)`

**Uses:**
- `plan:internal/shortcutcli#Mount`
- `plan:internal/shortcutcli#Load`
- `plan:internal/shortcutcli#Shortcut`
- `plan:internal/shortcutcli#GroupID`
- `internal/clihelp#Invocable`
- `cmd/lyx#run`

**Intent:**
D4 and D5: `lyx` mounts the shortcuts, and the CLI guards treat them as shortcuts, not modules.

- `newRoot` calls `shortcutcli.Mount(root, newRoot)` after `root.AddCommand(...)` and before the help command is set, so every root `newRoot` builds, the fresh roots a shortcut runs its argvs on included, carries the same shortcuts.
  Cobra lists the shortcut group under its `Shortcuts:` title in `lyx --help`, and the ungrouped module commands under its `Additional Commands:` title.
  The root's `Long`, inside `newRoot`, gains one line after "Available modules": "Shortcuts, from internal/shortcutcli's embedded file: attach, prime."
  The two file comments at the top of `cmd/lyx/main.go` (the file comment and the package comment, which say the root is assembled from every module's `Command()`) each gain that `newRoot` then mounts the shortcuts of `internal/shortcutcli`.
- `TestCLITree_EveryCommand`'s subtest "root.Long names every mounted module" skips a child whose `GroupID` is `shortcutcli.GroupID`, since a shortcut is not a module.
  The tree walk itself still visits the shortcuts, so `TestCLIForm_EveryCommand` checks their `Short`, `Use` and audience unchanged.
- `TestSandboxCoverage_AllModulesCoveredOrExcluded` skips a root child whose `GroupID` is `shortcutcli.GroupID` when it collects the registered modules, beside the `completion` skip, with a comment saying a shortcut runs other modules' commands and has nothing of its own for a sandbox scenario to exercise.
  `registration_test.go` needs no change: it finds packages by their `Command()` function, and `internal/shortcutcli` has none.

`cmd/lyx/shortcuts_test.go`:

- `shortcutProblems` returns one problem per defect of `shortcuts` against `root`: a name equal to a root command outside the shortcut group; a name a second shortcut also uses; an empty argv; an argv whose words, through `root.Find`, resolve to an error, to `root` itself, to a command that is not `Runnable` or not `clihelp.Invocable` (a bare group such as `reed`), or to a command in the shortcut group.
- `TestShortcuts_FileAndRootListing` (tier 1, parallel) builds `newRoot()` and:
  - asserts `shortcutProblems(root, shortcutcli.Load())` is empty, with `Load`'s error nil;
  - runs synthetic rows through `shortcutProblems` against the same root, each expected to report a problem containing a named substring: a shortcut named `reed`, two shortcuts named `go-to`, an empty argv, the argv `[reed]`, the argv `[nosuch, verb]`, and the argv `[prime]`;
  - runs `run([]string{"--help"}, &out)` and asserts exit 0 and that the output holds the `Shortcuts:` heading followed by an `attach` and a `prime` line before the `Additional Commands:` heading;
  - runs `run([]string{"help", "index"}, &out)` and asserts the operator index holds the line `` - `prime [<args>...]`: shortcut: lyx orch start, then lyx reed attach `` and the `attach` line.
- `cmd/lyx`'s row in `cmd/lyx/testdata/test-budget.yaml` goes from 78 to 79 for the new top-level test.

Docs:

- `docs/overview.md` gains, after the **help** bullet of the module list, a **shortcuts** bullet: `lyx attach` (`lyx reed attach`) and `lyx prime` (`lyx orch start`, then `lyx reed attach`) are top-level operator shortcuts declared in `internal/shortcutcli/shortcuts.yaml`, embedded in the binary and added only by a commit to that file; global verbosity flags reach every expanded command and other words the last; they are not modules. It points at the `internal/shortcutcli` package documentation.
  The package tree gains the line `internal/shortcutcli/` "the top-level operator shortcuts (`lyx attach`, `lyx prime`) from one embedded file".
- `pattern/PATTERN-cli-cobra.md`'s Package naming deviations gain: "`shortcutcli` mounts one root command per shortcut from its embedded file, with no `Command()` or `RunCLI`; a shortcut is an alias stretched to a sequence of argvs, so the module guards skip its group."

**ImpactSummary:** `newRoot` is the root of every `lyx` invocation, so the shortcut group appears in every root's tree; the two module guards that enumerate root children skip it, and the help and index output gain the group, with no assertion elsewhere on the old root help layout.

**Commit:** 5: lyx attach and lyx prime: mount the shortcuts on the root
