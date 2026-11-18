```yaml
lens: code-scope
outcome: reported
```

Task: the code-scope lens, which traces each change in the diff to the board entry's goal points and each goal point to the changes that build it.

**Decisions** (board entry `## Goal`, `_lym/discussion/decision-record.md:3-8`; the record has no `Added after Discussion` entries):
- G1: give `lym selfreport create` a `--body-file <path>` flag.
- G2: `--body-file` takes `-` for stdin.
- G3: refuse or warn when `-b` text is a single line naming an existing file.

**Per change → decision**
- C1, the `--body-file` flag and its read path (`cli.go:77`, the `resolveBody` `body-file` case at `cli.go:136-138`, `readBodySource` at `cli.go:143-161`) → G1 and G2.
- C2, `MarkFlagsMutuallyExclusive("body", "body-file")` (`cli.go:78`) → forced by G1. Two flags that both set the body need a rule for when both are given.
- C3, the refusal: `existingFileNamedBy` (`cli.go:163-175`) and the refusal branch in `resolveBody` (`cli.go:130-134`) → G3. The diff takes the "refuse" option.
- C4, the `stdinArgument` constant and moving body resolution out of `runCreate` into `resolveBody` (`cli.go:23-28`, `cli.go:85-95`) → forced by C1. The `-` stdin handling is now shared by both flags.
- C5, the `--help` examples in `Long` (`cli.go:56-64`) → forced by C1 and C3. The reflection stencil tells agents to follow the command's own guidance (`friction-template-reflection.md:52`).
- C6, package doc lines (`cli.go:6-8`) → forced by C1 and C3.
- C7, the `docs/overview.md` selfreport entry (`001-docs.diff:9-11`) → forced by C1 and C3, which made the old text out of date.
- C8, test rows and the `chdirToReportFile` helper (`cli_test.go:119-133`, `176-179`, `301-303`) → forced by C1, C2 and C3.
- C9, commit `3919e83f`, comment line breaks → forced by C1 through C6 (comment-lint compliance).

**Per decision → changes**
- G1: C1, C2, C5, C6, C7, C8.
- G2: C1 and C4, pinned by the "body file via stdin" row (`cli_test.go:177`).
- G3: C3, C5, C6, C7, C8.

**Findings:** none. Every change traces to a goal point or is forced by one, and every goal point is built by the diff. No `Pattern-Departure:` trailer appears in the range's commit messages, and none is needed.

Tool calls: none.
