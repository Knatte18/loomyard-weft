---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-affected-set
    location: tools/retro/retro.sh:62
    summary: retro.sh names _lym inline in its git commands, outside a variable block, against the PATTERN-lym-files single-declarer rule
    fix: In tools/retro/retro.sh, declare the record paths once in a variable block below `set -euo pipefail`, for example `USAGE_DIR=_lym/usage`, `SHED_DIR=_lym/shed` and `REVIEWS_DIR=_lym/reviews`. Then replace every inline `_lym/...` in the ls-tree and show calls (the usage ls-tree, the `$tag:_lym/shed/$slug/status.json` show and the reviews ls-tree) with those variables. The header comment may keep naming the files.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/retroengine/retro.go:717
    summary: a --until later than now stamps the retro in the future, so a later bare retro never covers the runs landing between the filing and that stamp
    fix: In internal/retrocli/cli.go parseWindow, refuse a --until later than now with a message naming the flag, the value and the way forward (omit --until, or give a date no later than today). Alternatively, clamp it in retroengine's Window.end to the earlier of Until and Now. Then add a refusal row, or a TestSelect row for the clamp.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/retrocli/cli.go:360
    summary: fileRetro has no test, so filing and releasing the task, the duplicate-slug, missing-tools and non-prime refusals, the merge of report-dir and board names, and the success envelope are all unpinned
    fix: Add one tier1 table-driven test in internal/retrocli over a hubforge hub, run through clihelp.ExecuteIn at the prime. Its rows are a bare retro over a landed run, which asserts the board entry is filed with a null status and checks the envelope's slug, runs and report_dir; a non-prime worktree refused; a repository with no tools/retro refused; and a board already holding the plan's slug refused.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/retroengine/retro.go:838
    summary: Plan.TaskFields, which makes the task a darn task on the docs profile with the research label, has no test
    fix: In internal/retroengine/retro_test.go, add an assertion to TestPlan_BodyFillsTheShippedStencil, or a small tier0 test, on Plan.TaskFields's recipe (shedrun.RecipeDarn), profile (TaskProfile), kind (boardengine.KindTask), labels ([TaskLabel]), slug and body.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: tools/retro/figures.jq:25
    summary: the producers-against-medians figure, with its medians and its outlier rule of at least three sessions at twice either median, is not pinned by any test
    fix: In tools/retro/retro_integration_test.go, give the fixture three or more main sessions of one row, with one at twice the median turns or cost, and assert the matching "Producers against their medians" line of figures.md, outlier included.
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/retroengine/retro.go:772
    summary: when the cap cuts between runs landed in the same second, the left-out run is never covered, since the next retro keeps only runs landed strictly after the stamp
    fix: In Select's cap branch, once runs[:MaxRuns] is cut, drop from the kept tail every run whose Landed equals the first left-out run's Landed, keeping at least one run, so a second is never split across retros. Then add a TestSelect row with two runs landed at the same time across the cut.
  - id: F7
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/retrocli/cli.go:392
    summary: the duplicate-slug refusal says to file again a second later, but a named or capped plan takes its stamp from a landing time, so the retry builds the same slug and is refused again
    fix: Reword the refusal in fileRetro to name a way forward that changes the slug. Wait a second and retry for a bare retro ending now; name a different set of runs, or give --since and --until, for a retro stamped by a landing.
  - id: F8
    severity: LOW
    class: consistency
    origin: lens:code-correctness
    location: internal/retroengine/retro.go:757
    summary: for named runs From is the oldest run's own landing, so the rendered body's sentence that the runs landed after From excludes the oldest run it then lists
    fix: In Select's named branch, set From to just before the oldest run's landing (runs[0].Landed.Add(-time.Second)). Alternatively, change the stencil line in contracts/stencils/retro/retro-template-task.md to say "from {{.from}}", so the sentence holds for both kinds of plan, and update the TestSelect named-runs row to match.
  - id: F9
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/retrocli/cli.go:342
    summary: no test pins parseWindow's success path, so swapping the --since and --until targets or parsing in a non-UTC zone fails no test
    fix: Pin parseWindow's success path, for example through the F3 test's bare-retro row with --since and --until whose envelope from and until are asserted. Or add a tier0 row to TestRetro_RefusesBadArguments's package that reaches the parsed window.
  - id: F10
    severity: LOW
    class: design
    origin: lens:code-tests
    location: tools/retro/transcript.jq:59
    summary: the gate failure read from a background task notification, and the done-false result, are not covered by the transcript fixture
    fix: In tools/retro/retro_integration_test.go, add a second lym gate test call whose result opens "Command was moved to the background", plus a queue-operation line carrying a task-notification with that tool-use id and a nonzero exit code. Raise the expected GateRuns and GateFails to match.
  - id: F11
    severity: LOW
    class: design
    origin: lens:code-tests
    location: tools/retro/figures.jq:24
    summary: the Gate-fixer sessions column is unpinned, because the fixture has no gate-fixer session and a broken title match still gives the asserted 0
    fix: In tools/retro/retro_integration_test.go, add a main session titled ly:run-a:gate-fixer, and assert the Runs row's Gate-fixer sessions cell as 1, adjusting the other cells it changes.
  - id: F12
    severity: LOW
    class: design
    origin: handler
    location: internal/retroengine/retro.go:746
    summary: a named run landed before the 60-day lookback is refused as never landed, which misstates the cause; the lookback is also untested
    fix: In Select's named branch, word the refusal to say the run has no landing within the last NamedLookback, and to give the way forward. Then extend TestSelect's never-landed row to assert the lookback date in the message.
  - id: F13
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: _lym/landing/summary.md:80
    summary: the change description claims the tools/retro test checks every table retro.sh writes, but the test never reads facts.jsonl or usage.jsonl
    fix: Either add facts.jsonl and usage.jsonl rows to the tables slice in tools/retro/retro_integration_test.go, or reword the Tests line of _lym/landing/summary.md to name the tables the test checks.
  - id: F14
    severity: LOW
    class: design
    origin: lens:code-affected-set
    location: tools/retro/retro.sh:96
    summary: the transcript figures read only Claude Code transcript directories, so a run on another agent engine is reported as having no transcript directory with no hint that its engine is the cause
    fix: In tools/retro/retro.sh, name the transcript layout in the header comment as Claude Code's, and record a run whose transcript directory is absent in missing.jsonl with what "transcript directory (Claude Code layout)". Alternatively, in contracts/stencils/retro/retro-template-task.md section 1, tell the writer that a run on another engine has no transcript figures and is read from its records alone.
---

### [BLOCKING] retro.sh names `_lym` inline

**Location:** `tools/retro/retro.sh:62`, `:66`, `:69`, `:79`

**Issue:** `PATTERN-lym-files`, under Single declarer, says only `internal/lymdirs` names `_lym` or `.lym` in a path, with one exception: a `tools/` script's variable block.
`retro.sh` writes `_lym/usage`, `_lym/shed/$slug/status.json` and `_lym/reviews` straight into its `git ls-tree` and `git show` calls.
`tools/forkaudit/forkaudit.sh` keeps such paths in variables (`$STATE_PATH`, `$PLAN_DIR`).
A listed constraint is violated, so the finding is BLOCKING.

### [MEDIUM] A future `--until` skips runs permanently

**Location:** `internal/retroengine/retro.go:717`, `internal/retrocli/cli.go:342`

**Issue:** `Window.end` returns `Until` unchanged, and `newPlan` stamps the slug with it.
`--until 2026-11-20`, filed on 2026-11-16, gives `retro-20261120-000000`, but the retro reads only what had landed by the filing.
The next bare retro starts after 2026-11-20, so runs landed from 11-16 to 11-20 are covered by neither retro.

### [MEDIUM] `fileRetro` is untested

**Location:** `internal/retrocli/cli.go:360`

**Issue:** `cli_test.go` covers only the argument refusals that come before cwd resolution.
None of these is covered by any test:

- filing through `UpsertTask` and releasing through `SetStatus(nil)`;
- the refusals for a non-prime worktree, a missing `tools/retro` and a duplicate slug;
- merging report-directory and board names before `LastRetro`;
- dropping slugless landings in `landedRuns`;
- the success envelope.

These are the board entry's core design points: the command files one darn task and releases it.

### [MEDIUM] `Plan.TaskFields` is untested

**Location:** `internal/retroengine/retro.go:838`

**Issue:** These fields are what makes the filing "one darn `docs` task": recipe darn, profile docs, kind task, the research label and the slug.
No test reads them, though the function is pure and testable at tier0.

### [MEDIUM] The median figure is unpinned

**Location:** `tools/retro/figures.jq:25`

**Issue:** The board entry names "turns and cost per producer against their medians" as a figure.
The integration test asserts only the Runs row, the column totals and the ledger line.
No test reaches the median computation or the outlier rule.

### [LOW] A tie at the cap loses a run

**Location:** `internal/retroengine/retro.go:772`

**Issue:** The cap sets `until` to the last kept landing, and the next retro keeps only runs landed strictly after that stamp.
A left-out run landed in the same second as the last kept one is never covered.
`sortByLanding`'s slug tie-break shows that equal landing times were expected.

### [LOW] The duplicate-slug way forward is wrong for stamped plans

**Location:** `internal/retrocli/cli.go:392`

**Issue:** The refusal says "file the retro again a second later".
A named-run or capped plan is stamped with a landing time, not with now, so the retry builds the same slug and is refused again.

### [LOW] The named-run `From` contradicts the stencil

**Location:** `internal/retroengine/retro.go:757`

**Issue:** For named runs, `From` is the oldest run's own landing.
The body reads "The runs landed after {{.from}}", which excludes that run, and then lists it.

### [LOW] `parseWindow`'s success path is untested

**Location:** `internal/retrocli/cli.go:342`

**Issue:** Only refusal rows exist.
Swapping the `into` pointers of `--since` and `--until`, or parsing outside UTC, fails no test.

### [LOW] The background gate-failure path is untested

**Location:** `tools/retro/transcript.jq:59`

**Issue:** The fixture's only gate call fails in the foreground.
The task-notification exit-code path and the done-false fallback are not reached.

### [LOW] The gate-fixer column is unpinned

**Location:** `tools/retro/figures.jq:24`

**Issue:** The fixture has no session whose title ends in `:gate-fixer`.
The asserted `0` holds even when the match is broken.

### [LOW] The named-run lookback refusal misstates its cause

**Location:** `internal/retroengine/retro.go:746`

**Issue:** `Window.Start` reads landings only for the last 60 days for named runs.
A run that landed 70 days ago is told it "has no landing ... since <date>: name only landed runs", although it did land.
The message gives no way to reach such a run.
No row asserts the lookback.

### [LOW] The change description overclaims the test

**Location:** `_lym/landing/summary.md:80`

**Issue:** The description says the `tools/retro` test checks "every table it writes".
It asserts `sessions`, `refusals`, `rows`, `keys` and `missing`, but never `facts.jsonl` or `usage.jsonl`.

### [LOW] Runs on other engines are reported as missing transcripts

**Location:** `tools/retro/retro.sh:96`

**Issue:** Transcripts are read only from Claude Code's `~/.claude/projects` layout.
A run on another engine lands in `missing.jsonl` as "transcript directory", and neither the script nor the stencil says why.

## Rejected

- code-affected-set F2 (the `retrocli` line missing from `PATTERN-cli-cobra`'s Package naming deviations): the deviation list covers a cli package that lacks its own engine or uses another module's kernel in place of one, and `retrocli` has `retroengine`. Cross-cli imports such as `reportcli` → `creelcli`, `burlercli` → `orchcli` and `configcli` → `fabriccli` are widespread and unlisted, so the list is no registry of cross-module imports.
- code-affected-set F3 as a `PATTERN-code-conventions` violation: `retro.sh` chooses no engine; it reads whatever transcripts exist, and the tools the board entry names (`tools/tokencount`, `tools/forkaudit`) read the same layout. It is kept as the LOW scope gap F14, not as a BLOCKING constraint violation.

## Handler review notes

Against the amended board entry, the decision record's other sections being empty:

- **The command.** No argument covers the runs since the last retro. Slugs or `--since`/`--until` are taken. One darn `docs` task is filed from the stencil and released. The runs, an exact cap of 12 moments and the question set are in the body.
- **The scripted part.** `retro.sh` plus `tokencount` and `forkaudit` produce each named figure: per-producer turns and cost against medians, serial single lookups, guard refusals, verify-gate runs and reworks, gate-fixer sessions, review rounds and recurring keys. A live run over two real landed runs exited 0 and rendered `figures.md`.
- **The LLM part.** The stencil carries the routing rule, with a new prompt sentence as the last resort.
- **The output.** The report is stamped under `docs/research/retro/`, proposals are ordered most severe first and each is traced to a run, session and turn. The run files no board notes.
- **The last retro.** It is the newest report-directory or board-entry stamp.
- **Friction-Reflect.** It is kept as is.
- **The rename.** `PrimeGeometry` leaves no reference to the old name, and its behavior is unchanged.
