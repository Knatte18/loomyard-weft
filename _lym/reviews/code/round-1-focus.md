---
round: 1
focus:
  - "internal/selfreportcli/cli.go `existingFileNamedBy`: does the guard catch the reported case (relative path such as `.scratch/x.md`, trailing newline or spaces) without refusing legitimate one-line bodies? Does `os.Stat` through symlinks or a path outside the working directory behave sensibly?"
  - "internal/selfreportcli/cli.go `resolveBody`: check `--body -` and `--body-file -` stdin handling, `--body \"\"` still giving an empty body rather than none, and `--body-file` naming an empty file or a directory."
  - "`MarkFlagsMutuallyExclusive` error text: the test asserts the literal `[body body-file]`; does the refusal reach the user through the shared envelope with exit code 1 and nothing sent to GitHub?"
  - "Decision record `## Goal`: the idea says refuse or warn; does the diff deliver `--body-file <path>` with `-` for stdin and the refusal, and license nothing beyond that (scope)?"
  - "Claims in `_lym/landing/summary.md` against the diff: directories and missing files filed as text, error wording, help examples, docs/overview.md and package doc."
  - "internal/selfreportcli/cli_test.go: the `chdirToReportFile` call added to every row of `TestRunCreate_Success` and `TestCommand_RefusedBeforeTransport`; does `t.Chdir` interact safely with the tests' other seams, and is any new row redundant with an existing one?"
---

Round 1 seed: a small diff in one package plus an overview doc line, so focus stays on the guard's edge cases and the scope match with the board entry.
