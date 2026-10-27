---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/lymcwd/docslink_test.go:560
    summary: the new subtest does not pin the bound that only a data/ directly under docs/research/<report>/ is skipped
    fix: In the subtest "research report data is skipped while the report itself is scanned", add the files "docs/other/data/x.md" and "docs/research/x/sub/data/x.md", each holding the same broken link "[p](pattern/PATTERN-x.md)", and change the expectation to three breaks, report.md, docs/other/data/x.md and docs/research/x/sub/data/x.md, in scan order, so a predicate widened to any data/ directory under docs/ or to any depth fails the subtest.
---

### [LOW] Skip bound is untested

**Location:** internal/lymcwd/docslink_test.go:560
**Issue:** The decision record bounds the skip to a `data/` directly under a report directory.
The subtest has one skipped path and one scanned path, so widening `docsLinkIsResearchData` to any `data/` under `docs/`, or to any depth under a report, still passes.
The predicate itself is correct: I ran `docs/research/x/data` (four parts), `docs/research/x/sub/data/a.md`, `docs/other/data/a.md` and `docs/research/data/a.md` through it, and all return false.

## Rejected

- Lens code-tests: no test pins that a link from a scanned file into a `data/` file still resolves. The diff leaves that resolution path untouched, so it adds no behavior to cover.

## Focus departures

None. Each focus question is settled:

- The path test matches exactly `docs/research/<report>/data/...` and nothing else, and a four-part path such as a file named `data` is not skipped.
- The subtest builds its fixture through `writeTree` under `t.TempDir()`, with no `testdata` in its paths. It fails if the skip is removed or widened to all of `docs/research/x/`.
- No `PATTERN.md` or `pattern/` text states the guard's scope, so the record's claim holds and the doc comments carry the exclusion.
- Only the source side is skipped, so resolution into `data/` files, anchors included, is unchanged.
- The allowlist is now an empty `[]scankit.Entry{}`. Its only reader is the repository scan, and the allowlist subtests use their own local slices.
