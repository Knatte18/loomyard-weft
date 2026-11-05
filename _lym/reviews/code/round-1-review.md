---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: docs/research/twill/twill-discussion-bench/deviations.md:1
    origin: lens:code-affected-set
    summary: deviations.md still names twill-discussion-bench.md as the full record, but the commit renamed that file to report.md in the same folder, so line 1 and line 5 point at a file that no longer exists
    fix: "In docs/research/twill/twill-discussion-bench/deviations.md replace the reference twill-discussion-bench.md on line 1 and the one on line 5 with report.md, the file that now holds the record beside deviations.md. Keep every other word of the file unchanged."
  - id: F2
    severity: LOW
    class: consistency
    location: docs/research/glyph/glyph-audit-2026-10-15/COMMANDS.md:13
    origin: lens:code-affected-set
    summary: The step-0 reproduction command still copies from the pre-move path docs/research/glyph-audit-2026-10-13/, while the same file's line 101 was rewritten to the new path, so the file treats the same old path two ways and the decision record's closing grep for old paths still finds this line
    fix: "In docs/research/glyph/glyph-audit-2026-10-15/COMMANDS.md line 13 change the source of the cp to the new folder docs/research/glyph/glyph-audit-2026-10-13/ (keeping the absolute checkout prefix and the brace list), so the setup step resolves and agrees with line 101. If the operator wants the line kept as the logged command of the old checkout, instead revert line 101 to the old path and add one sentence under the title saying the paths in this file are those of the old checkout."
  - id: F3
    severity: NIT
    class: design
    location: internal/lymcwd/docslink_test.go:565
    origin: lens:code-tests
    summary: The research-data subtest has rows for data/ at study depth and series depth but none for a data/ folder directly under a topic folder, which the predicate skips although a topic is not a study, so that boundary is unpinned
    fix: "In the subtest research study data at any depth is skipped while the report itself is scanned, add one file row docs/research/topic/data/x.md with the same link body. Add it to the expected breaks if the predicate should not skip a topic-level data folder (then change docsLinkIsResearchData so parts[3:len(parts)-1] starts one element deeper), or leave it out of the expected breaks to pin the current skip."
---

### [LOW] deviations.md cites a file name the commit renamed

**Location:** docs/research/twill/twill-discussion-bench/deviations.md:1
**Issue:** The commit moves `twill-discussion-bench.md` to `twill/twill-discussion-bench/report.md`. `deviations.md` sits in that folder and still says the full record is `twill-discussion-bench.md`, on lines 1 and 5. A search for basenames present at the base commit and absent now finds this as the only remaining prose reference to a renamed study file. The decision record asks that no reference to a moved path survive.

### [LOW] COMMANDS.md treats the same old path two ways

**Location:** docs/research/glyph/glyph-audit-2026-10-15/COMMANDS.md:13
**Issue:** Line 101 of this file, an orientation `cat`, was rewritten to `docs/research/glyph/glyph-audit-2026-10-13/`. Line 13, the step-0 `cp` that the file tells the reader to run in order to reproduce every table, still reads `docs/research/glyph-audit-2026-10-13/`, and that path no longer exists. The change description lists this line among the recorded data kept on purpose, but the rewrite of line 101 shows the file was not treated as a record throughout. The decision record's closing grep finds line 13.

### [NIT] No test row for a topic-level data folder

**Location:** internal/lymcwd/docslink_test.go:565
**Issue:** `docsLinkIsResearchData` now matches `data` anywhere in `parts[3:len(parts)-1]`. The subtest pins depth 3 (`topic/x/data`), depth 4 (`topic/series/x/data`), `docs/research/data/x.md` (scanned) and `docs/other/data/x.md` (scanned). `docs/research/topic/data/x.md` returns true, so its links are skipped, and no row says whether that is intended. No such folder exists today, so this is a boundary left open and not a live defect.

## Rejected

- Old paths in `calls.tsv` and `findings.tsv` under `docs/research` (lens:code-affected-set, handler): verbatim captures of what sessions ran or read, and rewriting them would alter measurement data; the change description names them as kept on purpose, and the decision record's goal is that references go stale, not that records change.
- `docs/quarry-holistic-fix-log.md:5` cites `docs/research/quarry-port-log.md` (lens:code-affected-set, considered): the file is absent both before and after the range, so the commit did not cause it.
- Script root lookup outside a git checkout (lens:code-correctness, considered): the old `../../..` form assumed the checkout layout too, so it is no regression, and the staged copy that runs outside the repository sets `SCRATCH` and makes no git call.
- The two new `vocabScanAllowlist` entries (focus item, handler): both moved notes name weft and warp (`docs/logging-inventory.md:5-6`, `docs/reference/weaving-glossary.md:34-72`), the entries are exact paths with no trailing slash, and they replace the exemption the files had under `docs/research/`, so they are needed and not a loosening of the scan.
- `docsLinkIsResearchData` false skip of an ordinary doc (focus item, handler): `docs/research/data/x.md` and `docs/other/data/x.md` stay scanned and the slice bounds hold for every length above 4; only the topic-level case in F3 is open.
- Index completeness and link resolution of `docs/research/README.md` (focus item, handler): every inline link resolves, every study folder and lone report is listed with a line, and the series folders list their members.
- Nothing deleted (focus item, handler): the range has no `D` entry, and the content edits inside renamed files are path and link rewrites only, plus the `README.md` to `report.md` renames the commit message and change description name.
