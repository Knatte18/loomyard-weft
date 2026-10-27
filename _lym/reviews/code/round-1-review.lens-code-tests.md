```yaml
lens: code-tests
outcome: reported
```

**Per behaviour**
- Skip of `.md` files under `docs/research/<report>/data/...` as link sources: pinned by the new subtest "research report data is skipped while the report itself is scanned" (`docslink_test.go:560-570`). The `data/prompts/step.md` link would be reported if the skip were removed. It would also be reported if the skip were narrowed to the wrong shape, because the break list must hold only the `report.md` entry.
- The sibling `docs/research/x/report.md` is still scanned: pinned by the same subtest, through the exact-equality `want` on the one break.
- Removed allowlist entry and the "no stale entry" outcome: pinned by the existing `repo` subtest (`:419-420`), which fails on any break or stale entry. No new test is needed.
- Boundary shape of `docsLinkIsResearchData` (a `data/` elsewhere under `docs/`, deeper inside a report, or a file named `data`): none. The fixture has one skipped and one scanned path, so a predicate widened to `docs/research/x/` as a whole is caught by the report.md assertion. A predicate widened to any `data/` under `docs/`, or to any depth, is not caught. The decision record's Bound calls this out as a decision. This is a coverage gap in the tests, but it is not a defect in what the diff added.
- Only the source side is skipped, so links from scanned files into `data/` files still resolve: none. The existing "relative link to existing file" and "missing file" subtests cover link resolution in general. None of them targets a `data/` file.
- Fixture hygiene: no `testdata` in the paths, and `writeTree` builds under the temp dir, as the decision record's Constraints require. The subtest is not parallel, but it matches the sibling subtests.

**Test economy**
- The new subtest does not duplicate any existing subtest. None of them covers the skip.
- It is a subtest of `TestEnforcement_MarkdownLinks`, not a per-helper test of `docsLinkIsResearchData`.
- No review-fix test and no stencil assertion was added.

**Findings**
1. LOW, `scope`, `docslink_test.go:560-570`: the subtest pins only one positive and one negative path, so the "directly under the report" bound is untested. A skip widened to every `data/` under `docs/`, or to any depth under a report, would pass. The fix is to add a second scanned file such as `docs/other/data/x.md` or `docs/research/x/sub/data/x.md` with a broken link, and expect it as an extra break.
2. LOW, `scope`, `docslink_test.go:560-570`: no test pins that a link from a scanned file into a `data/` file is still resolved, which the decision record states as a decision. The fix is to add a row in which `report.md` links to `data/prompts/step.md#missing` and the break is expected.

Tool calls (1):
- Grepped `docslink_test.go` for subtest names and `docsLinkScan` call sites. My context held the diff hunks but not the neighbouring subtests, so I needed them to check for duplicates.
