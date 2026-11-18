```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, name the test row that would fail if it broke.

**Per behaviour, the test that pins it**

1. `--body-file <path>` files the file's contents. Pinned by `TestRunCreate_Success` row "body via file" (`cli_test.go:176`).
2. `--body-file -` reads stdin. Pinned by `TestRunCreate_Success` row "body file via stdin" (`cli_test.go:177`).
3. `-b -` still reads stdin after the move into `resolveBody`/`readBodySource`. Pinned by the existing row "body via stdin" (`cli_test.go:175`).
4. `-b` text that is exactly an existing file's name is refused with the way forward, exit 1, nothing sent. Pinned by `TestCommand_RefusedBeforeTransport` row "body text naming an existing file" (`cli_test.go:301`).
5. `-b` text that mentions a file inside a longer line is still filed as text. Pinned by row "body text mentioning a file" (`cli_test.go:179`).
6. `-b` text naming a missing file is filed as text. Pinned by row "body via flag" (`cli_test.go:174`), since no file named `details` exists in the temp directory.
7. `-b` text that names an existing file with surrounding whitespace or a trailing newline, the `strings.TrimSpace` in `existingFileNamedBy` (`cli.go:166`), is refused, and the message quotes the trimmed path. **None.** Every refused row passes the bare `report.md`.
8. `-b` text naming an existing directory is filed as text, the `!info.Mode().IsRegular()` branch (`cli.go:171`). **None.**
9. `--body` and `--body-file` together are refused, exit 1, nothing sent. Pinned by row "body and body file together" (`cli_test.go:302`).
10. An unreadable `--body-file` fails with `failed to read --body-file`, nothing sent. Pinned by row "missing body file" (`cli_test.go:303`). A directory or other read error takes the same `os.ReadFile` error branch, so no separate row is needed.
11. `-b ""` gives an empty body rather than no `body` field. The diff rewrites this path, moving the `Changed` check into `resolveBody` and adding an empty-string early return in `existingFileNamedBy`. **None.** The gap predates the diff, but the diff rewrites the code that carries it.
12. The multi-line exclusion `strings.Contains(path, "\n")` (`cli.go:167`). No row could tell it apart: `os.Stat` of a name with an embedded newline fails anyway unless such a file exists. This is not a coverage finding.
13. The help text examples and `docs/overview.md` are static prose and exempt.

**Findings**

- **MEDIUM**, class `design`, `internal/selfreportcli/cli_test.go:301`. The trimming in the guard is untested. Nothing pins that `-b "report.md\n"` or `-b " report.md "` is refused, or that the refusal names the trimmed path. Dropping `strings.TrimSpace` at `cli.go:166` would pass every row, because `os.Stat("report.md\n")` fails and the text would be filed. A path with a trailing newline is what an agent's `$(cat …)` or a heredoc produces, and the focus names this case. Fix: add a refused row such as `-b reportFileName+"\n"` that expects `"pass --body-file " + reportFileName`.
- **LOW**, class `design`, `internal/selfreportcli/cli_test.go:179`. The directory exemption is untested. `existingFileNamedBy` refuses only regular files (`cli.go:171`), and `_lym/landing/summary.md:13` says a directory "is still filed as text". No row passes a directory name, so dropping the `IsRegular` check passes every row. Fix: add a `TestRunCreate_Success` row with `-b "."` that expects body `"."`.
- **LOW**, class `design`, `internal/selfreportcli/cli_test.go:174`. `-b ""` giving an empty body is untested. The doc comment at `cli.go:120` states it, and the diff rewrote the path that carries it (`cli.go:125-135`, `165-168`). No row passes `-b ""`, so a regression to an emptiness check in place of `Changed` would pass. Fix: add a success row with `-b ""` that expects body `new("")`.

Tool calls: none. My inherited context held the diff, both files whole and the summary.
