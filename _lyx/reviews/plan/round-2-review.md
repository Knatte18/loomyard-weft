---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/08-batten-notice-log.md
    summary: The plan rests on lyx cleanup (bc9ee7a02) being in the branch, but HEAD does not contain it, so card 8's claim is false, card 9 copies a cleanup skip that does not exist, and the battencli and orchcli budget rows start from counts HEAD does not have
    fix: >-
      Make the decision record's rebase constraint hold before any card is built. The orch or operator rebases the task branch onto a main that contains bc9ee7a02 before Webster begins; the fixer cannot do this itself and escalates it to its parent. Once the rebase is done, re-check every budget figure against the rebased tree. If the rebase does not happen, rewrite the cards against HEAD instead. In card 8, replace the sentence `The branch already contains lyx cleanup (bc9ee7a02).` with one saying the rebase must happen before this card, and stop citing `shedrun.RunsRoot`, `shedrun.ScratchRoot` and the lyx temp root, which do not exist at HEAD. Card 8's test-budget bullet becomes `internal/battencli` 64 → 65. In card 9, replace `resolvePersistentPreRun skips it as it skips cleanup` with the skip HEAD supports (cli.go:292 skips only the `batten` root), and change its budget bullet to `internal/battencli` 65 → 68. Card 10's budget bullet becomes `internal/orchcli` 26 → 27.
  - id: F2
    severity: BLOCKING
    class: scope
    location: _lyx/plan/12-run-window-close.md
    summary: Card 12 adds the integration-tagged termwindow_integration_test.go to internal/termwindow, which has no TestMain, so the package breaks PATTERN-test-isolation and cmd/lyx/tmuxisolation_test.go fails
    fix: >-
      On card 12, add `internal/termwindow/testmain_test.go` under **Create:**. In the Intent's Tests list, add a bullet saying this file declares a `TestMain` that runs the package through `tmuxkit.Main` (`PATTERN-test-isolation`), because the new integration-tagged file makes termwindow a tagged package. The termwindow budget row stays at 3, since `TestMain` is not counted.
  - id: F3
    severity: MEDIUM
    class: scope
    location: _lyx/plan/01-render-size-vocabulary.md
    summary: ValidateDisplay prescribes refusals for an unknown SizeKind, ShareNum below 0, ShareNum above ShareDen and MinRows below 1, but no named test row reaches any of them; only top-positional, top-sharing and ShareDen 0 are reached
    fix: >-
      In card 1's Tests bullet for `TestFixedHeightPinsMatchesRulesPlacedHeights`, add `wantErr` rows where `Rules` errors and the pins are nil for each of these displays. First, a below-parent display with an unknown `SizeKind` such as `"columns"`. Then three content sizes, each with an otherwise valid declaration: one with `ShareNum` -1, one with `ShareNum` 2 over `ShareDen` 1, and one with `MinRows` 0.
  - id: F4
    severity: LOW
    class: scope
    location: _lyx/plan/01-render-size-vocabulary.md
    summary: Card 1 puts top-anchored cells first in partitionByAnchor's output, but orderStack's doc comment in policy.go still says the stack is in insertion order so a newly added strand is the bottom-most, which is false for a top strand added after a below-parent one
    fix: >-
      On card 1, add `internal/reedengine/render#orderStack` under **Edit:**. In the Intent's Placement paragraph, add a sentence saying `orderStack`'s doc comment now describes the placed cells, top-anchored first and then the below-parent stack, each in insertion order. The comment no longer claims that a newly added strand is always the bottom-most.
  - id: F5
    severity: LOW
    class: scope
    location: _lyx/plan/05-statuspane-pane.md
    summary: Card 5 gives FormatMinutes no test because cards 7 and 9 assert its exact strings, but neither card names a row with an hour or a day duration, so the 1h05m and 2d03h branches are not covered
    fix: >-
      In card 9's `TestRenderOverview` bullet, say that the two-run row gives one run a stage time under an hour, which renders like `12m`. The same row gives the other run an age over a day, which renders like `2d03h`. In card 7's `TestRenderChecklist` bullet, say that the row reviewed twice sums to more than an hour, which renders like `1h05m`. That way the exact strings the two tests assert cover every FormatMinutes branch.
---

### [BLOCKING] The plan assumes `lyx cleanup` is in the branch, but HEAD does not contain it

**Location:** `_lyx/plan/08-batten-notice-log.md`, with `_lyx/plan/09-batten-overview.md` and `_lyx/plan/10-orch-runs-strand.md`

**Issue:**
The decision record's Constraints say "the branch is rebased onto a `main` that contains `lyx cleanup` (bc9ee7a02) before D8 is built".
Card 8 states "The branch already contains `lyx cleanup` (bc9ee7a02)".
That is false at HEAD: `git merge-base --is-ancestor bc9ee7a02 HEAD` exits 1, and no card carries the rebase.
Other parts of the plan only make sense in a tree that has that commit:

- `shedrun` has no `RunsRoot(` or `ScratchRoot`; `paths.go` has only `RunsRootRel`.
- No lyx temp root exists.
- `internal/battencli` has no `cleanup` verb, and `resolvePersistentPreRun` (`cli.go:291`) skips only the `batten` root.
  So card 9's "skips it as it skips `cleanup`" has nothing at HEAD to copy.
- The budget rows the cards edit do not match HEAD.
  `test-budget.yaml` and the actual top-level test count both say `internal/battencli` 64 and `internal/orchcli` 26.
  Cards 8, 9 and 10 instead edit 68 → 69, 69 → 72 and 27 → 28.

Card 9's arithmetic of three new tests is right, but every base it starts from is wrong at HEAD.
The answer to focus item 4 is therefore "the per-card deltas match, the starting values do not".
D8's guarantee that the log lies outside every directory `lyx cleanup` sweeps has nothing to be checked against until the rebase happens.

### [BLOCKING] termwindow gains a tagged test file but no `TestMain` through `tmuxkit.Main`

**Location:** `_lyx/plan/12-run-window-close.md`

**Issue:**
`PATTERN-test-isolation` says: "Every test package with an `integration`-, `tmux`- or `llm`-tagged test file runs its tests through `tmuxkit.Main`."
`cmd/lyx/tmuxisolation_test.go` enforces it, and its allowlist `allowedNoTmuxMain` is empty.
Today `internal/termwindow` holds only `termwindow.go` and `termwindow_test.go`, and neither declares a `TestMain`.
Card 12 creates `termwindow_integration_test.go` (`//go:build integration`) but no file declaring `TestMain`, so the isolation scan fails on the first card-12 build.
The proposed fix overlays cleanly: `validate-plan --overlay` with F2's and F4's target changes together reports no new non-informational finding.

### [MEDIUM] Four of `ValidateDisplay`'s refusals have no reaching test

**Location:** `_lyx/plan/01-render-size-vocabulary.md`

**Issue:**
The Intent says `ValidateDisplay` refuses these displays:

- `own-window`
- `top` with a kind other than content
- an unknown `SizeKind`
- `ShareDen < 1`
- `ShareNum < 0`
- `ShareNum > ShareDen`
- `MinRows < 1`

Card 1's `wantErr` rows cover top-positional, top-sharing and `ShareDen` 0, and card 2's `TestAddStrandLocked` rows repeat those three at the op boundary.
No named test reaches the unknown kind, the negative or oversized share, or a zero minimum.
The rubric's reachability rule makes each of those a finding.
They are filed once here because they are one table's missing rows.

### [LOW] `orderStack`'s doc comment goes stale

**Location:** `_lyx/plan/01-render-size-vocabulary.md`

**Issue:**
`policy.go:109` says `orderStack` "returns stack ordered by insertion … so a newly added strand is the bottom-most and no strand above it moves".
After card 1, `partitionByAnchor` hands it top-anchored cells first.
A top strand inserted after a below-parent strand is then first, not bottom-most.
Card 1 rewords `partitionByAnchor`'s comment, the `policy.go` file comment, `height.go`'s comment and `cellPlan.ordered`, but not this one, and it does not list `orderStack`.

### [LOW] `FormatMinutes`' hour and day forms are not covered by any named row

**Location:** `_lyx/plan/05-statuspane-pane.md`

**Issue:**
Card 5 gives `FormatMinutes` no test of its own.
It relies on the exact strings `TestRenderChecklist` and `TestRenderOverview` assert, which follows `PATTERN-test-economy`.
Neither card 7's nor card 9's row list names a duration of an hour or more, so the `1h05m` and `2d03h` forms the Intent prescribes can go unpinned.

## Rejected

- Focus item 1, the envkit nil-legal paths: no defect.
  Card 8 lists `internal/testkit/envkit#nilLegal` and the three paths `InnerRun.NoticeLog.Now`, `Land.Notices.NoticeLog.Now` and `Teardown.Notices.NoticeLog.Now`.
  These are exactly the nested paths `walkNil` would report.
  `RowNoticeDeps` reaches `shedrecipe.Env` only through `LandDeps.Notices` (`land.go:83`) and `TeardownDeps.Notices` (`deps.go:317`), and `InnerRunDeps` only through `Env.InnerRun`.
- Focus item 2, card 1's statement that `layout.go` is untouched apart from one comment: no defect.
  `placement` is declared in `layout.go:20`, the Intent says that file changes only `placement.strip`'s comment, and the **Edit:** list names `render#placement` for exactly that.
- Focus item 3, the minute-resolution formatter: no defect.
  `FormatMinutes` has one owner, `internal/statuspane`, created on card 5.
  Cards 7 and 9 both list `plan:internal/statuspane#FormatMinutes` under **Uses:**.
  The existing `loomcli.formatElapsed` and `boardengine.FormatAge` render different formats (seconds, one compact unit), so the plan does not copy either of them.
- Focus item 4, test granularity on card 9: no top-level test pins a single helper.
  `fitOverview` is reached as rows of `TestRenderOverview`, and the three new top-level tests match the delta of +3.
  The base value is wrong, as F1 says.
