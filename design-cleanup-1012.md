# Small cleanups: tokencount cutoff skew, audit reader globs, duplicate weft carve-out

`cleanup-1012` · Task · enhancement, fabric · running · PR-Gate

Three small, independent cleanups gathered from recent PR-Gate reviews and a follow-up read of the vocabulary scan.

## tokencount cutoff skew

Seen at the PR-Gate review of small-fixes-oct11 (PR #556, 2026-10-11): `tools/tokencount` now takes a cutoff from the slug's archive tags and skips main-session transcripts that start before it, with no flag to turn it off.
A tag whose commit date is skewed against the transcript timestamps could drop sessions silently; the report only says how many it skipped.

Idea: watch for this in the tokencount-full-runs measurements, and if it shows, add a flag to disable the cutoff or a tolerance window.

## Audit reader globs

From the webster-audit-gaps PR-Gate review (PR #528, 2026-10-10): the classifier now splits lyx, git and sed segments into shell words and bounds unquoted globs there, but the `readOnlyCommands` readers (`cat *`, `grep -r`) still accept unquoted globs and arbitrary flags, as before the task.
The classifier and `readOnlyCommands` live in `internal/fabricengine/refscanner.go`.

Also, an unquoted glob after `--` is admitted for `git log` and `git diff`; git treats it as a pathspec, but the reason lives only in the code, not in `doc.go`.

## Duplicate loomyard-weft carve-out

`internal/lyxcwd/enforcement_test.go` has two carve-outs that admit the bare token `loomyard-weft` in the repo-root files (`CLAUDE.md`, `README.md`) during the fabric-vocabulary tree scan in `fabricVocabularyFailures`.

- The first is `fabricvocab.BareTokenInRootDoc` (defined in `internal/fabricvocab/fabricvocab.go`), from commit c05fb1389.
  For a file in `vocabScanRootFiles` the scan uses it in place of `fabricvocab.BareToken`; it strips the records repository's proper name through `recordsRepoNamePattern` before checking.
- The second was added later by the `plan-gate-fixes` PR (commit 7b17ddfac): the `recordsRepositoryName` constant (`"loomyard-weft"`) and a `strings.ReplaceAll(text, recordsRepositoryName, "")` on the text of every file in `vocabScanRootFiles`, applied before `BareTokenInRootDoc` runs.

Keep one carve-out and remove the other, with no change in what the scan admits.
