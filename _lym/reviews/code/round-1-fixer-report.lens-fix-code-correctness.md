```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the F1–F6 fix through the code-correctness lens. I found no findings.

I ran each changed branch with one concrete input.

- **`findLanding` parse failure (F1):** a description of `no heading\n` makes `ParseData` fail, and `findLanding` returns `Landing{Missing: noParsableDescription + ": " + err}` with a nil error. `run` stores that landing, and `writeRecipes` lists it under "No landed commit". The new test row fails with the old behaviour, because the old error return hits `t.Fatal(err)`. The parse errors quote the heading with `%q`, so the reason cannot break the one-line "No landed commit" list.
- **`ChangedLines` root commit (F3):** `git show --numstat --format= --diff-merges=first-parent <sha>` on a root commit diffs against the empty tree, giving 2 for the two-line root. With the old `diff sha^1 sha`, `TestCodeRepoChangedLines` would fail on the root row, because the diff errors. The merge-commit path is not tested, but no caller in the change set relies on it.
- **`-last` below 1 (F4):** with no slugs, no `-since` and `-last 0` or `-last -1`, `run` refuses before `Getwd`, `finishedRuns` or `ReadSet` are touched. `TestRunRefuses` reaches that branch with an empty `ReadSet`. Named slugs and `-since` runs are unaffected, since both skip the check.
- **`-since` in `run` (F2):** the new integration run passes `-last 1 -since 2026-01-01` and expects `Skipped 3 ... done, live, solo`. With the `limit = 0` override removed, the loop would stop after `full` and the skipped list would be `done, live`. So the test pins the override. The fake sessions are touched at 20:00–23:00 UTC on 2026-01-01, so they sit on or after the local start of that day in any time zone.
- **`TestParseSince`:** its valid row matches `time.Local`, as `parseSince` does.
- **`TestWriteRecipes` (F6):** the unpriced setting is applied to `runs[6]` and `runs[7]`, which are the `o1` and `p1` runs. The `bare` run, with 0 lines, shows `0` and `n/a`. `odd` shows `$0.400 + unpriced`, and `only` shows `unpriced` in both cells. The gate showed the earlier index slip and then passed.
- **`tapestryRow` regexp (F6):** it needs `| 1 | 3 |` inside the `tapestry` row, so it is stricter than the two substring checks it replaces.
- **Callers:** `findLanding` and `ChangedLines` have no other callers. The `landingRepo` interface is unchanged. `fakeLandingRepo` is unaffected.
- **Wrapper headers (F5):** the `.sh` and `.cmd` wording match each other.

Further tool calls:
1. `grep` of `Errorf` and `errors.New` in `internal/summaryparser/summary.go`. My context lacked the parse error texts, which I needed to know whether the reason string can contain a raw newline.
