---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/retroengine/retro.go:192
    summary: Select panics with an index out of range when every landed run in the window shares the oldest landing's second and their count passes the cap
    fix: "In internal/retroengine/retro.go Select, bound the widening loop as `for cut < len(runs) && runs[cut].Landed.Equal(firstLeftOut) { cut++ }`. In internal/retroengine/retro_test.go TestSelect, add a row whose landed runs all share one second past the cap, for example landed runs a and b both at at(10,0,0), last at(7,0,0), hasLast true, a window with Now set to now and MaxRuns 1, and want a Plan with Slug retro-20261116-100000, From at(7,0,0), Until at(10,0,0) and Runs a and b."
  - id: F2
    severity: BLOCKING
    class: consistency
    location: tools/retro/retro_integration_test.go:72
    summary: The tools/retro test names the _lym records directory inline in fixture paths, which PATTERN-lym-files reserves to internal/lymdirs outside a tools script's variable block
    fix: In tools/retro/retro_integration_test.go buildRecords, import github.com/Knatte18/loomyard/internal/lymdirs and build every fixture path from lymdirs.LymDirName, as tools/forkaudit/forkaudit_integration_test.go does, for example filepath.Join(dir, lymdirs.LymDirName, "usage", "session-s1.json"), for the three usage records, the shed status and the two review facts files.
  - id: F3
    severity: LOW
    class: design
    location: tools/retro/retro_integration_test.go:72
    summary: The outlier fixture's only outlier passes both the turns and the cost threshold, so the test still passes when figures.jq's turns-or-cost rule becomes turns-and-cost or its threshold rises
    fix: In tools/retro/retro_integration_test.go buildRecords, set session s1's cost_usd to 0.6 and session s3's to 1.2, so s1 is an outlier on turns alone (6 against a median of 2) and s3 on cost alone, at exactly twice the median of 0.6. Update the usage.jsonl rows, the s1 and s3 sessions.jsonl rows, the Runs figure's cost cell to $2.3, and the producers line to list both outliers in the order figures.jq prints them, run-a s1 (6 turns, $0.6) and run-a s3 (1 turns, $1.2).
  - id: F4
    severity: LOW
    class: consistency
    location: _lym/landing/summary.md:77
    summary: The summary's Tests section lists only the argument refusals for internal/retrocli and omits the fileRetro integration test and the TaskFields test the diff adds
    fix: In _lym/landing/summary.md under Tests, extend the internal/retrocli bullet with the tier1 integration test that files and releases the task over a hubforge hub and covers the prime-only, missing-scripts, empty-window and duplicate-slug refusals and the success envelope, and add the task's board fields to the internal/retroengine bullet.
  - id: F5
    severity: LOW
    class: consistency
    location: contracts/stencils/retro/retro-template-task.md:5
    summary: The stencil tells the writer to write one report and change nothing else, then asks it to add a line to the research index
    fix: In contracts/stencils/retro/retro-template-task.md line 5, reword the sentence to name the index line as the one other change, for example Write one report and its line in the research index, and change nothing else, followed by the existing list.
---

### [BLOCKING] Select panics when the cap falls inside a second every run shares

**Location:** internal/retroengine/retro.go:192

**Issue:** The cap branch walks `cut` down to 0 while the runs before it share the first left-out run's second.
At 0 it widens with `for runs[cut].Landed.Equal(firstLeftOut) { cut++ }`, which has no upper bound.
When every run in the window landed in that one second, the loop reads `runs[len(runs)]` and panics.
Trace with two runs `a` and `b` landed at the same second and `--max-runs 1`: `firstLeftOut` is `runs[1].Landed`, `cut` drops from 1 to 0, then climbs to 2, and `runs[2]` is out of range.
The test row "a cap inside the oldest landing's second keeps that whole second" always has a later run `d`, so it never reaches this case.
`lym retro --max-runs 1` over two runs whose landing commits share a committer second crashes instead of filing.

### [BLOCKING] The tools/retro test names `_lym` inline

**Location:** tools/retro/retro_integration_test.go:72

**Issue:** PATTERN-lym-files, Single declarer: only `internal/lymdirs` names `_lym` or `.lym` in a path, and the one exception is a `tools/` script's variable block.
`retro.sh` now keeps its record paths in that block, but its Go test writes `_lym/usage/session-s1.json`, `_lym/shed/run-a/status.json` and `_lym/reviews/code/round-N-facts.md` as literals.
The sibling `tools/forkaudit/forkaudit_integration_test.go` builds the same kind of fixture from `lymdirs.LymDirName`.

### [LOW] The outlier fixture does not tell `or` from `and`

**Location:** tools/retro/retro_integration_test.go:72

**Issue:** figures.jq marks a session whose turns or cost reach twice its row's median.
In the fixture, s1 has 6 turns against a median of 2 and costs $1.5 against a median of $0.5, so it passes both tests with room to spare.
The asserted producers line stays the same if `or` becomes `and`, or if the factor of 2 rises to 3.
The test catches a broken median and a broken three-session floor, but not the rule the board entry asks for: producers against their medians on turns and cost alike.

### [LOW] The change description's test list is stale

**Location:** _lym/landing/summary.md:77

**Issue:** The Tests section says `internal/retrocli` covers "the argument refusals that come before any read".
The diff also adds `TestRetro_FilesAndReleasesTheTaskOfALandedRun`, which files and releases the task and reaches the prime-only, missing-scripts, parsed-window and duplicate-slug refusals.
It also adds `TestPlan_TaskFieldsFilesOneDarnTaskOnTheDocsProfile`, which the `internal/retroengine` bullet does not name.

### [LOW] The stencil's "change nothing else" contradicts its index step

**Location:** contracts/stencils/retro/retro-template-task.md:5

**Issue:** Line 5 reads "Write one report and change nothing else", and section 4 then has the writer add a line under a `retro` section of the research index.
A writer that follows line 5 literally skips the index, and the index is how the report is listed with the other research reports.

## Focus verdicts

- `tools/retro/retro.sh`: every `_lym` path, `_lym/usage`, `_lym/shed` and `_lym/reviews`, comes from the `USAGE_DIR`, `SHED_DIR` and `REVIEWS_DIR` block below `set -euo pipefail`; no git or file command writes one inline. Not a defect. The Go test is another matter; see F2.
- `--until` later than now: `parseWindow` refuses it, naming `omit --until`, and `TestRetro_RefusesBadArguments` reaches the refusal. Every other stamp is now, the last kept landing or the newest named landing, none of them later than now, so no future stamp can hide runs from the next bare retro. Not a defect.
- `fileRetro`: `TestRetro_FilesAndReleasesTheTaskOfALandedRun` reaches the non-prime refusal (naming the prime), the missing `tools/retro` refusal, the parsed `--since`/`--until` window through the empty-window refusal naming both ends, the success envelope's five keys, the filed task's null status, recipe, profile, labels and body, and the duplicate-slug refusal. Covered.
- `Plan.TaskFields`: `TestPlan_TaskFieldsFilesOneDarnTaskOnTheDocsProfile` pins slug, title, kind, recipe, profile, labels and body. Covered.
- figures.jq and transcript.jq: the gate-fixer column is asserted as 1 in the Runs line and would read 4 if the title filter broke. The background-task failure path is reached through call `tf` and its queue-operation notification, so `gate_fails: 2` would drop to 1 without it. The notification's shape matches real transcripts on this machine (`<tool-use-id>toolu_…` and `Background command failed (exit code 1)`). The outlier rule is reached but only partly pinned; see F3.
- `Select`: the cap branch is wrong when one second holds every run; see F1. A named retro's `From`, one second before the oldest named landing, agrees with the stencil's "landed after {{.from}}". The lookback refusal names the slug, the 60 days, the window's start and the `--since`/`--until` way forward, and the misspelt-slug case, and `TestSelect` asserts each part.
- The summary against the tools/retro test: every table the summary lists (`sessions`, `refusals`, `rows`, `facts`, `keys`, `missing`) and `usage.jsonl` is read and compared whole; `figures.md` is checked by selected lines. Holds.
