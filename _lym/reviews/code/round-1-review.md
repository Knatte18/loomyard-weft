---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: internal/selfreportcli/cli.go:77
    summary: The new flag name body-file is a raw literal repeated in four places, and body gains two more repeats, against PATTERN-code-conventions' named-constant rule
    fix: In internal/selfreportcli/cli.go, declare two constants beside stdinArgument, bodyFlag = "body" and bodyFileFlag = "body-file", each with a one-line doc comment. Use them in place of every literal flag name in Command (the StringP and String flag definitions and the MarkFlagsMutuallyExclusive call) and in resolveBody (both Changed calls and both GetString calls). Leave the shorthand "b" and the help and usage text as they are.
    origin: lens:code-affected-set
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/selfreportcli/cli_test.go:301
    summary: No test pins that -b text naming an existing file with surrounding whitespace or a trailing newline is refused, so dropping the TrimSpace in existingFileNamedBy passes every row
    fix: In TestCommand_RefusedBeforeTransport in internal/selfreportcli/cli_test.go, add a row named "body text naming an existing file with padding" with args create, T, -b, " " + reportFileName + "\n", wantExit 1 and wantOutput "pass --body-file " + reportFileName. No new test function.
    origin: lens:code-tests
  - id: F3
    severity: LOW
    class: design
    location: internal/selfreportcli/cli_test.go:179
    summary: No test pins that -b text naming an existing directory is filed as text, or that -b with an empty string files an empty body rather than none
    fix: In TestRunCreate_Success in internal/selfreportcli/cli_test.go, add two rows. Row "body text naming a directory" passes create, T, -b, "." and expects wantBody new("."). Row "empty body via flag" passes create, T, -b, "" and expects wantBody new(""). Both use withNumber, title T, labels bug and number 123. No new test function and no new fixture.
    origin: lens:code-tests
  - id: F4
    severity: NIT
    class: design
    location: internal/selfreportcli/cli.go:133
    summary: The refusal's way-forward prints the path unquoted, so a path holding spaces gives a suggested command the shell splits
    fix: In resolveBody in internal/selfreportcli/cli.go, change the refusal's format from "pass --body-file %s to file the file's contents" to "pass --body-file %q to file the file's contents". In TestCommand_RefusedBeforeTransport, change the "body text naming an existing file" row's wantOutput to the quoted form, `pass --body-file "` + reportFileName + `"`, and the F2 row's likewise.
    origin: lens:code-correctness
---

### [BLOCKING] Flag names repeated as raw literals

**Location:** internal/selfreportcli/cli.go:77

**Issue:** `PATTERN-code-conventions` says a name that can go stale is one named constant, declared once and referenced everywhere, never a repeated literal (`pattern/PATTERN-code-conventions.md:11`).
A cobra flag name is such a name: when one occurrence changes and another does not, `Changed("body-file")` reports false and the flag is silently ignored.
The diff adds `"body-file"` at `cli.go:77`, `78`, `136` and `137`, and adds two more `"body"` repeats at `cli.go:78`, `125` and `126` beside the pre-existing one at `76`.
Several CLI packages in the repository already name their flags as constants, such as `internal/hibernatecli/cli.go:18` and `internal/machinecli/cli.go:26`.
The same diff names the stdin value `"-"` as `stdinArgument`.

### [MEDIUM] The guard's trimming is untested

**Location:** internal/selfreportcli/cli_test.go:301

**Issue:** `existingFileNamedBy` trims the text before `os.Stat` (`cli.go:166`), so `-b "report.md\n"` is refused.
A trailing newline is what `$(cat …)` or a heredoc gives an agent, and it is the case the round's focus names.
Every refused row passes the bare `report.md`, so removing `strings.TrimSpace` leaves the suite green while the reported failure returns for padded paths.

### [LOW] Directory and empty-body behaviors are untested

**Location:** internal/selfreportcli/cli_test.go:179

**Issue:** `existingFileNamedBy` refuses only a regular file (`cli.go:171`), and `_lym/landing/summary.md:13` claims a directory is still filed as text; no row passes a directory, so dropping `IsRegular` passes every row.
`resolveBody`'s doc comment (`cli.go:120`) promises that `--body ""` gives an empty body rather than none, and the diff rewrote the path that keeps that promise; no row passes `-b ""`.

### [NIT] Unquoted path in the way-forward

**Location:** internal/selfreportcli/cli.go:133

**Issue:** The refusal quotes the body text with `%q` but prints the path with `%s`.
For a path holding a space, the suggested `--body-file` argument pasted into a shell splits into two words.

## Rejected

- Assistant F1 at BLOCKING: its coverage gaps are kept as F2 and F3. Its escalation to BLOCKING rests on `PATTERN-test-speed` requiring boundary coverage, but that entry's declared text asks that a test pin behavior no other test covers, not that every boundary have a row. The gaps take the rubric's ordinary severity.
- Assistant F1's empty `--body-file` row: an empty file takes the same `os.ReadFile` and pointer path as a non-empty one, which "body via file" pins. No distinct branch is left uncovered.

## Review notes

Scope matches the board entry: `--body-file` with `-` for stdin, and a refusal for one-line `-b` text naming an existing regular file.
The refusal reaches the user through `clihelp.RunRootCtx`'s envelope with exit 1, and no request is sent.
The mutually-exclusive refusal takes the same path, and the `[body body-file]` row pins it.
Using `t.Chdir` in the serial tables is safe, since no test or subtest calls `t.Parallel`, and the file header names the working directory as shared state.
The claims in `_lym/landing/summary.md` match the code.
`PATTERN-refusal-way-forward` does not bind selfreport, so the refusal needs no `way forward:` clause.
