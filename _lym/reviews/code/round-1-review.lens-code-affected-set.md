```yaml
lens: code-affected-set
outcome: reported
```

Task: code-affected-set lens. List is per change: the set found, covered or not.

**Per-change list**

1. **Global `--envelope` persistent flag** (`internal/clihelp/readable.go:20`, `cmd/lym/main.go`).
   - Set found:
     - `lym creel read` local `--envelope` bool (`internal/creelcli/cli.go:683`).
     - `lym loom arbiter` local `--envelope` string path (`internal/loomcli/arbiter.go:98`).
     - `metaFlags` in `internal/clihelp/jsonhelp.go:47`.
     - `pattern/PATTERN-cli-cobra.md:18-19`, which documents the global `--json` flag and its shadowing.
     - `shortcutcli`'s test root and `status_test` (both call `InstallJSONHelp`).
   - Covered: partly.
     - The local flags shadow the global one, since cobra's flag merge skips an existing name.
     - In `UseReadableOutput`, `Lookup` finds the local flag and the `Changed` check keeps the envelope when it is set. That is correct for `creel`.
     - The arbiter's string flag needs a value, so its `Changed` is true whenever it is used, which is also correct.
     - A bool `--envelope` flag on the root's own JSON help would be listed, because `metaFlags` excludes only `json` and `help`. I read nothing that shows this is wrong.
   - Not covered: `PATTERN-cli-cobra.md` is not updated (finding F1).

2. **`exitState.finish` and `onFinish`, run by `RunRootCtx` before the cobra error envelope** (`internal/clihelp/exec.go:47-66, 193-197`).
   - Set found:
     - Every `RunRootCtx` caller: `RunRoot`, `Execute`, `ExecuteIn`.
     - The `exitState` struct, which is only built in `NewExitContext`.
     - `exec_test.go`.
   - Covered:
     - `es` is never nil, because `NewExitContext` always returns it.
     - With no registered steps the order is unchanged.
     - A caller that never calls `UseReadableOutput` sees no change.

3. **The root pre-run now calls `UseReadableOutput`, so `cmd.SetOut` swaps the writer** (`cmd/lym/main.go:170`).
   - Set found:
     - 185 `OutOrStdout()` call sites in non-test code.
     - `internal/clihelp/index.go` (no `envelope` hits).
   - Covered by the swap: `OutOrStdout` call sites.
   - Not covered by the swap: three direct `os.Stdout` uses.
     - `internal/reedcli/attach.go:130` sets `attach.Stdout = os.Stdout`.
     - `internal/loomcli/checklist.go:122` and `internal/battencli/overview.go:539` read the terminal size from `os.Stdout`.
     - These are interactive-handoff or size lookups, not envelope output, so they do not bypass the summary.
   - `cmd/lym/main.go:63` does `root.SetOut(os.Stdout)` before the pre-run, so `OutOrStdout()` returns an `*os.File` there, and `WriterIsTerminal` works as intended.
   - A module that captures its writer before the pre-run is not reached. I found none in the searches I ran.

4. **Docs: `docs/overview.md`, the `cmd/lym/main.go` Long text, the `clihelp` package doc.**
   - All three are updated and consistent with the diff.
   - `docs/overview.md:353` still says "All commands print JSON" and then adds the terminal exception, which is fine.

5. **Stencils and skills.**
   - The diff changes none, and `summary.md` explains why: an agent's output is a pipe, not a terminal.
   - The board entry's Open question ("whether agents need the flag added") is answered in `summary.md` by the terminal rule.
   - The rubric's stencil-wording check finds no weakened rule.

6. **PATTERN entries.**
   - No `PATTERN.md` declaration is in the directive, so there is nothing to land.
   - `PATTERN-cli-cobra` is touched in substance (see F1).

7. **Commit messages.**
   - No `Pattern-Departure:` trailer appears, and none is needed.

**Findings**

- **F1 — LOW, class `consistency`.**
  - Location: `pattern/PATTERN-cli-cobra.md:18-19`, evidence at `internal/clihelp/readable.go:20`.
  - Summary: the diff adds a second global flag, `--envelope`, declared by `clihelp.InstallReadableOutput`, but the background file for `PATTERN-cli-cobra` documents only the global `--json` flag and its shadowing rule.
  - The background file also says nothing about the operator-audience terminal summary.
  - Fix: add a bullet after line 19 saying `--envelope`, declared by `clihelp.InstallReadableOutput`, keeps the full envelope where an operator verb on a terminal prints its summary, and that a command's own local `--envelope`, such as `lym creel read`, shadows it.

- **F2 — NIT, class `consistency`.**
  - Location: `internal/clihelp/jsonhelp.go:47`, evidence at `readable.go:20`.
  - Summary: `metaFlags` still lists only `json` and `help`.
  - The global `--envelope`, a persistent flag on the root, therefore appears in the root's `lym --json` flag list, while no leaf lists it.
  - I did not run the JSON help, so this is not confirmed.
  - Fix: add `"envelope": true` to `metaFlags` only if the root's output shows it. A command's own `--envelope` (`creel read`, `loom arbiter`) is then also hidden from its help, so adding it is wrong unless the exclusion is limited to `jsonHelpValue`-style global flags.
  - Recommend verifying before acting.

**Tool calls**
1. Grep for `InstallJSONHelp`/`increase log verbosity` files, global-flag docs, and flag handling in `jsonhelp.go`: my context lacked the set of files that document or wire global flags.
2. Read `jsonhelp.go:44-52`, `pattern/PATTERN-cli-cobra.md:10-30`, the `shortcutcli` test root, and the `index.go` envelope grep: my context lacked `metaFlags`, the pattern's text on global flags, and how test roots are wired.
