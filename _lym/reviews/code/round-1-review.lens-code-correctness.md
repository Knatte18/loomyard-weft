```yaml
lens: code-correctness
outcome: reported
```

Task: review the correctness of the changed code (lens: code-correctness).

**Per function or branch, case run**

- `docsLinkIsResearchData("docs/research/x/data/prompts/step.md")`: it splits into 6 parts, `len > 4`, and parts 0, 1 and 3 match, so it returns true. Correct.
- `docsLinkIsResearchData("docs/research/x/data")`: this is 4 parts, the boundary case, because the file is named `data`. `len > 4` is false, so it returns false and the file is scanned. Correct. Only a `.md` file named exactly `data` could reach this, and the walk selects only `.md` files, so the case cannot occur.
- `docsLinkIsResearchData("docs/research/x/report.md")`: 4 parts, so false. Correct.
- `docsLinkIsResearchData("docs/research/x/sub/data/a.md")`: `parts[3]` is `sub`, so false. Correct. A `data/` deeper inside a report is still scanned, as the decision record bounds it.
- `docsLinkIsResearchData("docs/other/data/a.md")`: `parts[1]` is `other`, so false. Correct.
- `docsLinkIsResearchData("docs/research/data/a.md")`: 4 parts, so false. Here `data` is the report name, not a `data/` directory. Correct.
- `docsLinkIsResearchData("")`: the split gives one empty part, so false. Correct.
- Path form: `f.Rel` is `filepath.ToSlash` of the path relative to the walk base (`scankit.go:183-187`), so the predicate sees slash-separated base-relative paths on every platform. Correct.
- The early `return` in the `docsLinkScan` walk callback: it skips only the outgoing-link extraction for that file. The file still counts in `scanned`, since `Walk` returns `len(files)`. Links from scanned files into `data/` files are still resolved, because resolution reads the target tree and not the source filter. No state is left inconsistent. Correct.
- The empty `docsLinkAllowlist`: with no entries, the stale check has nothing to report and the break filter has nothing to match. The `[]scankit.Entry{}` literal is type-correct. Correct. The removed entry's file is now skipped, so no break is produced for it.
- The new subtest, with `data/prompts/step.md` and `report.md` both holding a broken link: the fixture goes through `writeTree` and its paths hold no `testdata`, so `scankit` skips nothing. If the skip were removed, 2 breaks would be reported and the subtest would fail. If the skip were widened to all of `docs/research/x/`, 0 breaks would be reported and it would fail. The `Line: 1` and `Reason: "missing file"` fields in `want` are not verified against `docsLinkBreak` here (see the unverified note below). Correct.

**Findings:** none.

**Unverified:** I did not confirm the exact `Reason` string `"missing file"` or the `docsLinkBreak` field names, because the manifest does not show them. A mismatch would make the subtest fail, and the verify gate would catch it.

**Tool calls**
1. `grep` of `scankit.go` for `Rel`/`testdata`: to confirm `File.Rel` is slash-normalized and base-relative, and that `testdata` directories are skipped, which the manifest lacked.
