```yaml
lens: fix-consistency
outcome: reported
```

No findings.

- **Raw flag names:** a repo-wide grep finds none left in `selfreportcli`. The `"body-file"` literals in `internal/boardcli/cli.go:680` and `:1111` belong to a different command and are outside this fix.
- **Quoted refusal path:** the only place that asserts the `pass --body-file` text is `cli_test.go:304-305`. Both rows now expect the `%q` form with JSON-escaped quotes, and the gate passed.
- **Help text:** `Long` in `cli.go` says text naming an existing file is refused and to pass `--body-file`. That is still accurate and names no path, so the quoting change does not touch it.
- **Doc comments:** the package doc, the `stdinArgument` comment and the `resolveBody` comment still match the code. The new `bodyFlag` and `bodyFileFlag` comments are consistent with them.
- **Test header and rows:** the header comment in `cli_test.go` still holds. The new directory and empty-body rows in `TestRunCreate_Success` and the padding row in `TestCommand_RefusedBeforeTransport` sit in the tables the review named.

Tool calls:
1. `grep` for `pass --body-file`, `"body-file"` and `Changed("body` across the repo, to find twins outside the files already in my context.
