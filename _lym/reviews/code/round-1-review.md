---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: tools/tokencount/landed.go:88
    summary: A run whose archived change description does not parse aborts the whole report instead of being listed as having no landed commit
    fix: "In findLanding (tools/tokencount/landed.go), when summaryparser.ParseData fails return Landing{Missing: reason} with nil error, where reason is a new const such as `noParsableDescription` plus the parse error text, so writeRecipes lists the run under `No landed commit`. Change the TestFindLanding row `a change description that does not parse` (landed_test.go) to expect that Landing and no error."
  - id: F2
    severity: MEDIUM
    class: scope
    origin: lens:code-tests
    location: tools/tokencount/main.go:283
    summary: The -since path in run (limit forced to 0, the active since header) and a valid parseSince date have no test
    fix: "Add a -since run to TestRunReadsArchivedShedStatus (time_integration_test.go) that asserts the header `finished tapestry and darn runs active since <date>`, and add a parseSince row for a valid YYYY-MM-DD date and for the empty value; the existing refusal row in TestRunRefuses stays."
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: tools/tokencount/landed.go:257
    summary: A title-matched commit with no first parent makes git diff fail on sha^1, and that aborts the whole report
    fix: "In codeRepo.ChangedLines (tools/tokencount/landed.go) diff against the empty tree when the commit has no parent, or have findLanding return Landing{Missing: reason} when the diff fails, so one such run does not abort the report. Add a root-commit case to TestRunReadsArchivedShedStatus or a fake-repo row to TestFindLanding."
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: tools/tokencount/main.go:282
    summary: recentRuns now treats last <= 0 as unlimited, so -last 0 or a negative -last without -since counts every finished run, where it used to count none
    fix: "In run (tools/tokencount/main.go) refuse a -last below 1 when -since is empty, with a message naming the flag, and keep limit = 0 only for the -since case. Add a row to TestRunRefuses (main_test.go)."
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: tools/tokencount/tokencount.sh:9
    summary: Both wrapper headers still say the report opens with the most expensive parts and the cost per run, then give each segment's cost, and omit the Recipes section the diff added after the cost per run
    fix: "In the header comment of tools/tokencount/tokencount.sh and of tools/tokencount/tokencount.cmd, add the recipe comparison after `the cost per run` in the sentence that starts `The report opens with`; keep the two wrappers worded the same."
  - id: F6
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: tools/tokencount/recipes.go:563
    summary: The zero-line branch of writeRecipes and the unpriced branches of Cost.precise have no test row, and the integration check of the changed-line count matches two loose substrings
    fix: "Add to TestWriteRecipes (landed_test.go) a darn run that landed 0 lines, expecting `0` changed lines per run and `n/a` per line, and a run with an unpriced cost, expecting `unpriced` and `$X.XXX + unpriced`. In TestRunReadsArchivedShedStatus replace the two substring checks with the whole `| tapestry | 1 | ... | 1 | 3 | ... |` row."
---

### [MEDIUM] A change description that does not parse aborts the whole report

**Location:** `tools/tokencount/landed.go:88`, `tools/tokencount/main.go:331`
**Issue:** `findLanding` returns an error when the description archived at a run's tag fails `summaryparser.ParseData`, and `run` returns it, so one run with an empty or headingless description loses the whole report. The other two missing-landing cases (no description at the tag, no titled commit) become a `Landing.Missing` reason and the report goes on. The entry asks to compare recipes over a period, and the report is the only output, so one old run should be listed and not fatal. `TestFindLanding` pins the error today.

### [MEDIUM] The -since path in run is untested

**Location:** `tools/tokencount/main.go:283`
**Issue:** `TestRecentRuns` pins the `since` argument of `recentRuns`, and `TestRunRefuses` pins the refusal of a bad date. Nothing runs `run` with a valid `-since`, so the override of `-last`, the `active since` header and `parseSince` of a real date can break unseen. `time_integration_test.go` only passes `-last 1`.

### [LOW] A landed commit with no first parent aborts the report

**Location:** `tools/tokencount/landed.go:257`
**Issue:** `ChangedLines` diffs `sha^1` against `sha`. For a root commit `sha^1` does not resolve, `gitexec.Run` fails, and `findLanding` returns the error, which ends the report. Rare for a landing, but the same all-or-nothing failure as the parse case.

### [LOW] -last 0 now counts every finished run

**Location:** `tools/tokencount/main.go:282`, `tools/tokencount/count.go` (`recentRuns` loop)
**Issue:** `recentRuns` now takes `last <= 0` as no limit so that `-since` can reuse it. A plain `-last 0` or `-last -1` used to end in the no-runs error and now counts everything. The flag text says nothing of it.

### [LOW] Wrapper headers omit the Recipes section

**Location:** `tools/tokencount/tokencount.sh:9`, `tools/tokencount/tokencount.cmd:9`
**Issue:** The diff added `-since` to both wrappers and changed the sentence that follows, but the sentence naming what the report opens with still lists only the most expensive parts and the cost per run. The two wrappers do match each other, and they match the new `-since` usage.

### [LOW] Zero-line and unpriced branches are untested

**Location:** `tools/tokencount/recipes.go:563`, `tools/tokencount/cost.go:63`
**Issue:** `writeRecipes` guards `lines > 0` for the per-line figure, so a recipe that landed commits of zero changed lines prints `n/a`; no row reaches it. `Cost.precise` has three branches and only the priced one is exercised. The real-git integration check asserts the 3 changed lines through the substrings `| tapestry | 1 | ` and ` | 1 | 3 | `, which would also match another row or column.

## Rejected

- Lens code-tests F4, a `TestFindLanding` row for a commit just before the span start and rows for failing fakes: the slack is one symmetric expression and the failing-fake paths only wrap and return the error, so a row adds nothing a reader of the code would act on.
- Lens code-tests note that the `recentRuns` inclusive edge of `since` is not pinned: the `every run since a time` row sits 150 minutes from the nearest run, and a boundary row would pin one comparison operator; not worth a finding.
- Lens code-tests note that `-code` and `Calibrate` receiving the `-code` repository have no test: with no flag the default is the working directory, as before, and the wiring is one assignment.
- Handler check of the squash-off landing: with `squash` false a landing leaves the task commits and no commit titled after the description, so the run lands in the `No landed commit` list. The code's doc names the squash default as the assumption and the section reports the run, so it is not a defect.
- Handler check that Tapestry runs still count the same: the only cost change is that `describe` moves from the Other segment to Landing, a total-neutral regrouping the diff states in `segmentDefs` docs and `TestPartOf`.

The claims of `_lym/landing/summary.md` were checked against the diff and each holds, bar the gaps filed above (the parse failure and the `-last 0` change are not mentioned in it). The three commit messages carry no `Pattern-Departure:` trailer, and the declared PATTERN changes list is empty.
