# Card 8 — tender-command

**Edit:**
- `internal/pausecli#Command`
- `internal/pausecli/doc.go`
- `internal/pausecli#TestPauseAndResume_WriteLiftAndRefuseOverTheHub`
- `internal/pausecli#TestPauseAndResume_ApplyTheCallerRules`
- `cmd/lyx#newRoot`
- `cmd/lyx#TestShortcuts_FileAndRootListing`
- `internal/shortcutcli/shortcuts.yaml`
- `internal/shortcutcli/doc.go`
- `docs/overview.md`
- `tools/sandbox/SANDBOX-CORE-SUITE.md`
- `pattern/PATTERN-cli-cobra.md`

**Rename:**
- `internal/pausecli#ResumeCommand` -> `internal/tendercli#resumeCmd`

**Create:**
- `internal/tendercli#pauseCmd`
- `internal/tendercli#statusCmd`

**Intent:**
The pause module becomes the `tender` module's one cobra subtree, with `lyx pause` and `lyx resume` kept as shortcuts (D1, D2, D3, D4).
The package names stay `pausecli` and `pauseengine` until card 12 renames them, so every glyph here resolves against the tree as it stands.

`Command()` returns one root command, `tender`, a group with no `RunE` and no audience annotation.
It prints help on a bare call and refuses a mistyped subcommand, as `PATTERN-cli-cobra`'s group rule says.
Its subcommands are built in `cli.go`:
- `pauseCmd()` is today's `pause [<slug>] [--grace <seconds>]` with its sequence, caller rule, inputs and envelope unchanged (D4).
  It loses the `--status` flag, with no alias, and its `Long` loses that paragraph and the example.
  `lyx tender pause --status` is then refused by cobra as an unknown flag.
- `resumeCmd()` is today's `ResumeCommand`, renamed to unexported, since it is now built only by `Command` and no longer mounted as a root command.
  Its doc comment loses "the second root command of the pause module".
- `statusCmd()` is `status`, with no arguments, which runs `pauseengine.Status` and prints its report under `status`, as `lyx pause --status` did.
  It takes no caller check, so any caller, an agent included, may run it, since it writes nothing (D3).
  Card 9 adds the sessions and the stale listing to the report.
Each subcommand keeps the operator audience annotation and a lower-case `Short`.
`revive` arrives in card 10.
`RunCLI` and `RunCLIIn` keep running `Command()`.

`newRoot` mounts `pausecli.Command()` once and drops the `pausecli.ResumeCommand()` line and its comment.
Its `Long` names `tender` in place of `pause, resume` in the "Available modules" list, which `cmd/lyx/clitree_test.go` checks against the mounted modules.
`shortcuts.yaml` gains `pause: [tender, pause]` and `resume: [tender, resume]`.
`internal/shortcutcli`'s routing already forwards every word after the shortcut name to the last argv, so `lyx pause <slug> --grace 10` runs `lyx tender pause <slug> --grace 10`.
Refusal texts and stencils that name `lyx pause` or `lyx resume` keep doing so, since the shortcut is the stable operator spelling.

`TestShortcuts_FileAndRootListing` asserts the operator index lines `` - `pause [<args>...]`: shortcut: lyx tender pause `` and `` - `resume [<args>...]`: shortcut: lyx tender resume ``.
Its `shortcutProblems` check already fails a shortcut whose name equals a root command outside the shortcut group.
So it also pins that `lyx --help` lists no top-level `pause` or `resume` module beside the shortcuts.

`TestPauseAndResume_WriteLiftAndRefuseOverTheHub` runs every verb through `Command()` with the subcommand as its first argument: `pause`, `status` and `resume`.
It reads the status report from the `status` subcommand run in the pair's worktree.
It gains the row "`pause --status` is refused as an unknown flag and adds no entry".
`cli_test.go`'s header comment names `lyx tender pause`, `status` and `resume` in place of `lyx pause --status`.
`TestPauseAndResume_ApplyTheCallerRules` runs its pause and resume rows through `Command()` with the `pause` and `resume` subcommands, and gains the row "an agent runs `status` and it succeeds".

Every doc names the new spelling:
- `pausecli/doc.go` names `lyx tender pause`, `lyx tender resume` and `lyx tender status`, and the shortcuts.
- In `docs/overview.md`, the module bullet becomes **tender**, with the verbs and the shortcuts and with `lyx tender status` in place of `lyx pause --status`.
  The shortcuts bullet and `shortcutcli/doc.go` list `lyx pause` and `lyx resume`.
- In `SANDBOX-CORE-SUITE.md`, S12's `**Covers:**` line names `tender`, since `cmd/lyx/sandbox_coverage_test.go` refuses a covered name no registered module carries.
  Its watch line asks whether `lyx tender status` lists the one live entry and changes nothing, and its title stays.
- `PATTERN-cli-cobra.md`'s deviation line says `pausecli` uses `pauseengine` and mounts one root command, `tender`.
No doc or stencil names `lyx pause --status` after this card.

**ImpactSummary:** The pause module's two root commands become one `tender` group; the only production caller of the renamed `ResumeCommand` is `newRoot`, which drops the call, and the operator spelling `lyx pause`/`lyx resume` survives through the shortcut file.

**Commit:** 8: tender: one module root with pause, resume and status; pause and resume stay shortcuts
