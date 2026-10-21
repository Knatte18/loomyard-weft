# Decision record — shellguard-fixes

## Goal

The shell guard in `internal/shellguard` refuses what it is meant to refuse and nothing else.
A refused go run is matched on the command that runs, never on text inside quoted arguments, grep patterns or heredoc bodies.
A shell redirect write to a project file is refused under the edit directive, like the other scripted-edit cases.

## Scope

- In: the shared command-position prefix every command-shaped row builds on, the raw go row's patterns, a new redirect-write row, the claude engine's rendering of row patterns, the package doc and the tests pinning each row.
- Out: a shell parser or a Go-side hook binary; the guard stays grep -E patterns over the PreToolUse payload.
- Out: writes through `tee`, `cp`, `dd`, a script or `bash -c`; the board entry names heredoc and redirect writes only.

## Decisions

### Command position is found by a quote-aware scan from the start of the command field

Decision: the command-position prefix anchors at the payload's `"command"` key and consumes the command left to right as shell units: an unquoted character, a JSON escape pair, a single-quoted string, a double-quoted string, and at most one heredoc operator after which the scan stops at the end of that line.
A row's command word matches only directly after a separator reached that way.
Every row built on the prefix gains it, not only the raw go row.

Rationale: the false denies of issue #770 come from text inside quotes, grep patterns and heredoc bodies, and a scan that stays outside quotes and before heredoc bodies is the closest grep -E gets to matching the command that runs.
One prefix keeps one notion of command position across the table.

Bound: the scan sees nothing inside quotes, after a heredoc operator's line, or behind an escaped double quote inside a double-quoted string it misreads, so a command there passes; the rows were guardrails before and stay guardrails.

### The raw go row's variable names match only where the shell reads or sets them

Decision: the slot, strand and prebuilt-binary variable names are refused when named outside quotes or expanded inside double quotes, and no longer when named inside a single-quoted or double-quoted grep pattern or a heredoc body.
The steer says a command setting or reading one is refused.

Rationale: a grep for the name runs nothing, and quoting it is the natural way to search.

Bound: `printenv` with the name in quotes, a heredoc body that expands one, and an expansion behind a misread double quote pass.

### A new row refuses redirect writes outside scratch paths

Decision: a row named `redirect-write`, installed for every run with no config key, refuses a `>`, `>>`, `>|` or `&>` redirect, reached by the quote-aware scan, whose target word is not under `.scratch/` by its relative path, `/tmp/` or `/dev/`.
A file-descriptor duplication such as `2>&1` and a target starting with a variable pass.

Rationale: grep cannot tell a tracked file from an untracked one, so the row treats every path outside the scratch places as a project file; the edit directive already sends every project-file write through Edit or Write.

Bound: it lets through `tee`, `cp`, `dd`, a write behind `bash -c` or a script, a redirect after a heredoc operator's line, a target behind a variable, and an absolute path into `.scratch/`; it falsely denies an untracked file outside the scratch places and a `>` comparison inside `[[ ]]` or `$(( ))`.

### Patterns may hold a single quote, and the engine quotes them

Decision: the claude engine shell-quotes each pattern into its hook command instead of wrapping it in bare single quotes, and the package doc's payload contract says a pattern may hold any character.

Rationale: the quote-aware scan has to name the single quote.

## Constraints

- `PATTERN-shell-guard-table`: the guard lives only in `internal/shellguard`, and every engine renders every row.
- `PATTERN-leaf-packages`: `internal/shellguard` keeps its import set.
- `PATTERN-no-denied-recovery`: the new row's steer names no denied command as a way forward.
- Steer and notice text hold no single quote, double quote or backslash, and the notice no line break.

## Auto-mode assumptions

- `/tmp/` counts as a scratch place for the redirect row, since it holds no project file, though the edit directive asks for `.scratch/`.
- The orch gets the redirect row like every other run, as it gets the in-place edit row.

## Open risks

- A grep without a DFA engine could run the quote-aware pattern slowly on a large heredoc payload.
- An agent that writes an untracked file outside `.scratch/` through a redirect is now refused and has to use Write.

## Acceptance criteria

- `grep` with a quoted pattern naming a module-wide or tmux-tier go test, or one of the variable names, passes the raw go row.
- A heredoc whose body holds a go test line or a variable name passes the raw go row.
- A commit message in double quotes naming a module-wide go test passes the raw go row.
- Every command the raw go, python, in-place edit, computed-rm and machine-sleep rows denied before, outside quotes and heredoc bodies, is still denied.
- `cat >> docs/x.md <<EOF`, `cat <<EOF > docs/x.md`, `echo x > file`, `printf x >> /abs/path/file` and `cmd &> out.log` are denied by the redirect row.
- `echo x > .scratch/a`, `cmd > /dev/null 2>&1`, `cmd > /tmp/x`, `cmd 2>&1 | tee` and `grep "a > b" f` pass the redirect row.
- The claude engine renders a pattern holding a single quote into a hook command that greps that pattern.
