```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens. For each change in the diff, find what reads or describes it and say whether the diff left any of it wrong.

**Per change, the set found and whether it is covered**

1. **New `--body-file` flag, made mutually exclusive with `--body`** (`internal/selfreportcli/cli.go:77-78`).
   - `cmd/lym/jsonhelp_test.go:113-124` lists `--body` and `--label` in `wantFlags`. Both flags are still present, so the test stays correct, and it does not have to name the new flag. Covered.
   - The `--help` text (`cli.go:56-64`) carries the flag. Covered.
   - `docs/overview.md:621-622`: covered.
   - The package doc (`cli.go:6-8`): covered.
   - `contracts/stencils/friction/friction-template-reflection.md:52` sends the agent to the command's own guidance for its fields. It names no body flag, so nothing in it goes stale. Covered.
   - `contracts/orchrole/orch-role.md:347`, `internal/friction/doc.go:49` and `cmd/lym/agentfiledissues_test.go` name only the verb. They are unaffected.
   - `docs/getting-started.md:41` and `internal/shellguard/rows_integration_test.go:287` mention a `--body-file`, but it belongs to the board and to gh. They are unrelated.
2. **The refusal of `--body` text that is one line naming an existing file** (`cli.go:130-134`).
   - The help text, `docs/overview.md:622` and the package doc describe it. Covered.
   - `PATTERN-refusal-way-forward` binds only webster, shed and loom (`pattern/PATTERN-refusal-way-forward.md:3-4`). selfreport is not bound, so no `way forward:` clause is required. No finding.
3. **The read-failure message `failed to read --body-file: …`** (`cli.go:156`).
   - The only reader is the new test row (`cli_test.go:303`). Covered.
4. **Error text for `--body -` on stdin, moved from inline code into `readBodySource`.**
   - The text is still `failed to read stdin: …`, and nothing outside the package asserts it. Covered.
5. **Repository rules.**
   - `PATTERN-github-auth`: there is still one filing path, through `selfreportengine.CreateIssue` in `runCreate`. Covered.
   - `PATTERN-test-speed`'s parallel rule: the tests do not call `t.Parallel`, and the file header names the shared state, now including the working directory (`cli_test.go:8-9`). Covered.
   - `PATTERN-code-conventions`, named constants: one finding, below.
6. **Declared PATTERN changes.** My directive declares none, so there is nothing to check.
7. **Commit messages.** Neither commit in the range carries a `Pattern-Departure:` trailer, so there is nothing to judge.

**Findings**

- **Severity:** BLOCKING. A violation of a listed constraint is BLOCKING; the handler may judge whether a cobra flag name counts as "a name that can go stale".
  - **Class:** consistency.
  - **Location:** `internal/selfreportcli/cli.go:77-78, 136-137`.
  - **Summary:** The diff repeats the flag name `"body-file"` as a raw literal in four places, against `PATTERN-code-conventions`, which requires one named constant for a name that can go stale. Those places are `Flags().String`, `MarkFlagsMutuallyExclusive`, `Changed` and `GetString`. It does the same with `"body"`, adding two more uses (`MarkFlagsMutuallyExclusive` and the `Changed`/`GetString` in `resolveBody`) to a literal the old code already repeated. The diff does name the stdin value `"-"` as `stdinArgument` (`cli.go:24`), so the convention was in view.
  - **Evidence:** `pattern/PATTERN-code-conventions.md:11` ("a name or path that can go stale: one named constant, declared once and referenced everywhere, never a repeated literal"), and `cli.go:76-78, 125-126, 136-137`.

**Tool calls**

1. Grepped `docs/getting-started.md`, `plugins`, `contracts` and `internal/friction/doc.go` for `selfreport`. My context had only a list of matching file names, not the lines that describe the body flags.
2. Grepped `internal/shellguard/rows_integration_test.go` and `docs/getting-started.md` for `body-file`. Earlier grep output named those files without showing whether they referred to selfreport's flag.
