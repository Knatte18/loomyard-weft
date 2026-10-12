# Card 4 — shortcutcli-package

**Create:**
- `internal/shortcutcli/shortcuts.yaml`
- `internal/shortcutcli/doc.go`
- `internal/shortcutcli/shortcut.go`
- `internal/shortcutcli#GroupID`
- `internal/shortcutcli#shortcutFile`
- `internal/shortcutcli#Shortcut`
- `internal/shortcutcli#Load`
- `internal/shortcutcli#parse`
- `internal/shortcutcli#Mount`
- `internal/shortcutcli#mount`
- `internal/shortcutcli#newShortcutCmd`
- `internal/shortcutcli#routedWords`
- `internal/shortcutcli#routeWords`
- `internal/shortcutcli#runArgvs`
- `internal/shortcutcli/shortcut_test.go`
- `internal/shortcutcli#TestShortcut_Run`
- `internal/shortcutcli#TestMount_File`
- `internal/shortcutcli#fakeRootFactory`
- `internal/shortcutcli#fakeRecorder`
- `internal/shortcutcli/testmain_test.go`

**Edit:**
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
D4: the package that declares the top-level operator shortcuts in one embedded file and mounts one root command per shortcut.
This card builds the package on its own; card 5 mounts it from `newRoot`.

`shortcuts.yaml` (embedded into `shortcutFile` with `//go:embed shortcuts.yaml`) maps a shortcut name to one argv, a list of words, or to a list of argvs run in order.
An argv names lyx words only, never a shell string: a command path, then any args and flags of that command.
The shipped file holds `attach: [reed, attach]` and `prime: [[orch, start], [reed, attach]]`, with a leading comment in semantic line breaks saying what the file is, that each value is one argv or a list of argvs, and that a shortcut is added only by a commit to this file.

`shortcut.go`:

- `Shortcut` has `Name string` and `Argvs [][]string`; `Load` returns `parse(shortcutFile)`, the shortcuts sorted by name.
- `parse` decodes the YAML into a mapping of names to `yaml.Node`s: a sequence of scalars is one argv, a sequence of sequences of scalars is a list of argvs, and any other shape (a scalar, a mapping, a mixed sequence, a document that is not a mapping, malformed YAML) is an error naming the shortcut.
  It does not reject an empty argv or an unknown command: card 5's test against the real root does.
- `Mount(root, newRoot)` calls `mount(root, newRoot, shortcutFile)`.
  `mount` parses; on an error it logs one `logger.Warn` (`"shortcutcli: the shortcut file does not parse, so no shortcut is mounted"`, with `err`) and mounts nothing.
  Otherwise it adds the group `&cobra.Group{ID: GroupID, Title: "Shortcuts:"}` to `root` and one `newShortcutCmd` per shortcut.
- `newShortcutCmd` builds `Use: "<name> [<args>...]"`, `Short: "shortcut: lyx <argv 1>, then lyx <argv 2>"` (one `lyx <argv>` per argv, joined by `, then `; so `shortcut: lyx reed attach` for `attach`), `GroupID: GroupID`, `DisableFlagParsing: true`, `Args: cobra.ArbitraryArgs`, and the annotation `clihelp.AudienceAnnotation: clihelp.AudienceOperator`.
  It declares no flags of its own.
  Its `RunE` returns at once on `clihelp.ShouldAbort`, then calls `routeWords(args)`:
  a help request sets the root's global `json` flag to the json word's value when one was given (`cmd.Root().PersistentFlags().Set("json", value)`, whose `clihelp` value also raises `--help`), prints `cmd.Help()` and runs nothing;
  otherwise it builds the argvs, the routed verbosity words prepended to every argv and the trailing words appended to the last, and calls `runArgvs`.
- `routeWords` walks the words up to the first `--`.
  Each word that starts with `-` is parsed alone against a mirror `pflag.FlagSet` of the root's three global flags, declared as the root declares them: a count `verbose`/`v`, a bool `help`/`h` and a bool `json` whose `NoOptDefVal` is `"true"`.
  A word that parses with no positional left is global: a verbosity word (`-v`, `-vv` and longer runs, `--verbose`, `--verbose=<n>`, `-v=<n>`) joins `Global`, and a word that sets `help` or `json` (`-h`, `--help`, `--help=<b>`, `--json`, `--json=<b>`, a shorthand run such as `-vh`) sets `Help`, recording the json value when it named `json`.
  Every other word joins `Trailing` verbatim in order: a flag no global flag spells (`-vx`, `--name`), a global flag with a value pflag refuses (`--json=bogus`, `--verbose=x`, `-h=bogus`), which D4 names in neither exception, and `--` with every word after it.
  Such a bad-value word fails in the last argv's root with that root's flag error, after the earlier argvs ran, as typing the expanded commands in turn would.
- `runArgvs` runs each argv in order on a fresh root from `newRoot`: `SetArgs(argv)`, `SetIn(cmd.InOrStdin())`, `SetOut(cmd.OutOrStdout())`, `SetErr(cmd.ErrOrStderr())`, then `clihelp.RunRootCtx(cmd.Context(), inner, cmd.OutOrStdout())`.
  The context carries the cwd the outer root was given, so the shortcut resolves nothing itself (`PATTERN-cwd-resolution`).
  A non-zero code stops the sequence and is returned; the `RunE` carries it with `clihelp.SetExit` and prints nothing of its own, since the inner root already printed its envelope.

Bound: a shortcut runs only lyx commands of the same binary, each through a full root with its own pre-run, refusals and pause holds, so it adds no path past any command's guard; no hub, repo or user can add or change one, since the file is embedded.
A shortcut's last argv can receive a local `--json` of its own only after `--`.

The root's persistent pre-run runs once for the shortcut command and once per argv in one process, and each repeat is harmless: trace adoption is behind `traceOnce` and re-exports the same id, `logger.Arm` captures its header once, `pausecli.InstallNetworkGate` swaps in an identical gate, and stencil seeding writes only a subtree that is due, so the repeat after the first pass finds nothing due and writes nothing.

`doc.go` is the package doc: the file and its two shapes, the mount from `newRoot`, the word routing (verbosity words to every argv, help and json words print the shortcut's help and run nothing, everything else and everything after `--` to the last argv), the fresh-root dispatch with its exit code, the parse-failure Warn, the bound above, and that shortcuts are not modules: they have no `Command()` or `RunCLI` and are skipped by the module guards in `cmd/lyx`.

`testmain_test.go` runs the package under `tmuxkit.Main`, as every package does.

Tests (tier 1):

- `fakeRecorder` records, per run of a fake leaf, its command path, positional args, the root's verbosity count, its local `--name` flag value, `lyxcwd.CwdFrom(ctx)` and, for `reed attach`, what it reads from `cmd.InOrStdin()`; it scripts a per-path exit code and envelope.
  `fakeRootFactory` returns a factory that builds a root like `newRoot`'s shape: `Use: "lyx"`, `clihelp.InstallJSONHelp`, the persistent count `verbose`/`v`, groups `orch` and `reed` with `clihelp.GroupRunE`, and leaves `orch start` and `reed attach` (the latter with a local `--name` string flag), each recording into `rec` and writing `output.Ok`, or `output.Err` with its scripted code through `clihelp.SetExit`.
- `TestShortcut_Run` (table, parallel) mounts the shortcuts `attach: [reed, attach]` and `prime: [[orch, start], [reed, attach]]` through `mount` on a factory root and drives it with `clihelp.ExecuteIn` from a temp cwd. Rows:
  `attach` runs `reed attach` once, with its envelope in the output and exit 0;
  `prime` runs `orch start` then `reed attach`, both with the shortcut's cwd;
  `prime` with `orch start` scripted to succeed with an already-running envelope still runs `reed attach`;
  `prime` with `orch start` scripted to fail with exit 2 runs no `reed attach`, exits 2, and the output holds exactly one error envelope;
  `prime --name x extra` gives `reed attach` the name `x` and the arg `extra`, and `orch start` no args;
  `-v prime`, `-vv prime`, `prime -v` and `prime --verbose=2` run both commands with verbosity 1, 2, 1 and 2;
  `prime --json`, `prime --json=true`, `prime --help`, `prime -h` and `prime -vh` run nothing and exit 0, the `--json` rows printing JSON help whose `name` is `lyx prime` and the others the text help carrying the shortcut's `Short`;
  `prime -- -h` runs both, `reed attach` receiving the positional `-h` and no help;
  `prime --json=bogus` runs `orch start`, then fails in `reed attach`'s root with exit 1 and exactly one error envelope, `reed attach`'s body not running;
  `attach` with the outer root's input set to a reader holding `typed` gives `reed attach` that input: the fake leaf records what it reads from `cmd.InOrStdin()`.
- `TestMount_File` (table; not parallel, since `logcapture.Capture` swaps the process-global logger output) calls `mount` on a fresh factory root with file bytes:
  the shipped `shortcutFile` mounts `attach` and `prime` in group `GroupID` with the `Use`, `Short` and audience above and `DisableFlagParsing` set, and logs nothing;
  malformed YAML, a scalar value, a mapping value and a sequence mixing a word and a list each mount no command, add no group and log exactly one Warn.

`cmd/lyx/testdata/test-budget.yaml` gains the row `internal/shortcutcli: 2`.

**ImpactSummary:** A new package with no caller until card 5; the only existing file touched is the test budget, which gains one row.

**Commit:** 4: shortcutcli: top-level operator shortcuts from one embedded file
