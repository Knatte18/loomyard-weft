---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: contracts/stencils/gatefix/gatefix-body-card.md:28
    summary: "The prescribed single-test re-run `-run '^(<its name>)$'` passes while running nothing when the failed test is a subtest, since go test does not split a pattern at a slash inside parentheses and the gate exempts a -run call from its empty-run check."
    fix: "In gatefix-body-card.md line 28, replace the re-run form with one that anchors each level of the test's path apart: split the reported name at each `/` and pass `-run '^<level1>$/^<level2>$...'`, for example `-run '^TestX$/^case$'` for `TestX/case`, a top-level test staying `-run '^TestX$'`. Add one sentence telling the fixer that a filtered run passes even when it matched no test, so it reads the run's output and counts the re-run as checked only when the failed test is shown to have run and passed."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: contracts/stencils/gatefix/gatefix-body-card.md:28
    summary: "The fixer is told to re-run any failure that is not one named test \"by the command the report names for it\", but the failure report names no command for a build, vet tierN, package-level (TestMain, panic, timeout) or lint failure, and line 29 bars the whole call, so the fixer has no sanctioned way to re-check those."
    fix: "Replace the clause \"and any other failed step by the command the report names for it\" in gatefix-body-card.md line 28 with a rule per failure kind, keyed by the run label the report prints: a `build` or `vet tierN` failure is re-run as `lym gate test --tier <N> <its package> -- -run '^$'`, which compiles the package under that tier's command and runs no test; a package failure outside any test is re-run as `lym gate test --tier <its tier> <its package>` with no filter; a `lint comments` or `lint pattern` failure is re-run as `lym loom lint-comments` or `lym loom lint-pattern`. Unverified: that every hub tier command compiles the named package before its -run filter applies; read the template tier commands in internal/gateslot/template.yaml before you write the build and vet rule, and name the tier's own command there if one does not."
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: contracts/stencils/gatefix/gatefix-body-card.md:28
    summary: "The re-run form has no -C, so a failed test in a nested module, reported by import path, runs in the tier's root directory and the go command refuses the package, where the red card call ran that module in its own directory."
    fix: "In the same line 28 rule, add that a failed test whose package lies in a nested module is re-run with `-C <its module directory>`, the worktree-relative directory the report's qualified file paths open with, and its package given relative to that module."
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/gatecli/cli.go:89
    summary: "The test command's long help still says a narrower scope under the go scoper runs the //lym:guard tests outside the scope, with no --gate card exception, contradicting the new behaviour and the --card flag help at cli.go:125."
    fix: "In cli.go line 89, add the exception to the sentence, for example: \"--guards, and a narrower scope while <scoper key> is go bar a --gate card call's, also run the //lym:guard tests outside the scope; --card adds tier-skipped ones.\" Keep the keyName(scoperKey) concatenation as it is."
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/gatecli/scope.go:76
    summary: "The scope.guardStep field comment still says a --card scope under the go scoper plans a guard step, which a --gate card scope no longer does."
    fix: "Change the comment at scope.go line 76 to: \"guardStep means the scope form plans a guard step: a computed, --base or --card scope under the go scoper, bar a --card scope under the card key.\""
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/gatecli/scopes_integration_test.go:826
    summary: "No test pins two behaviours the change description states for --gate card: that the vet step over the cards' own packages still runs, and that --guards still forces a guard step; the vet rows run --card with no key and every --guards test uses explicit targets."
    fix: "In TestGateTest_Scopes, add an assertion to the existing vet row near scopes_integration_test.go:741 that runs the same call with `--gate card` and expects the own packages (`./lib` and `.`), the vet invocations that row already expects, and no guardInvocation. In the `--gate card` row at :826, add a call with `--guards` that expects `[]string{\"./lib\"}`, `[]string{\".\"}` and then guardInvocation. Read the vet row's existing wantArgs before writing it, since its vet invocation values are not quoted here."
  - id: F7
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/landing/summary.md:64
    summary: "The Merge danger section calls the door two-way without naming that the change alters what the hub's gate.gates.card setting (scope card) means, from the cards' packages and their dependents with a guard step to the cards' packages alone with none, as template.yaml's comment now states."
    fix: "Under `## Merge danger` in _lym/landing/summary.md, keep or change the door call, and add one line naming the meaning change of gate.gates.card's scope card under the card key, why the call still holds (no config value is rewritten, nothing is pushed or deleted, and the previous binary reads the same file the old way), and widen the blast radius to name the Webster and Weaver implementer and card-fixer prompts the diff also changes."
---

Round 1 reviews `af381392785e9c90c2721d86abf86fb8e12f8dab..HEAD`, two commits, against the board entry in `_lym/discussion/decision-record.md`.
The record holds no `Added after Discussion` entries.
The scope change in `computeScope` is correct for every case the lenses and I traced: the root and nested modules, `--main-packages`, a card with no Go package, a package not yet created, the vet step, the watch skip, a forced `--guards`, and the dotnet and none scopers, which return before the branch.
The Webster and Weaver flows, recovery included, render a `--gate batch` call over the same cards after the card calls.
That call's tier0 run takes the dependent packages, and its guard step scans every module guard test the listed runs leave out, tier1 guards included.
The verdict is APPROVED, with two MEDIUM defects in the new fixer prompt.

### [MEDIUM] A subtest re-run passes while running nothing

**Location:** `contracts/stencils/gatefix/gatefix-body-card.md:28`

**Issue:** The failure report names a failed test by its full path, subtests included (`internal/gatecli/counters.go:89`).
Go's `splitRegexp` (`$GOROOT/src/testing/match.go:181-197`) splits a `-run` pattern only at a `/` outside parentheses.
So `^(TestX/case)$` is one element, matched against the top-level name `TestX`, and no test runs.
`emptyTierFailure` exempts any call whose words after `--` carry `-run` (`internal/gatecli/tierrun.go:754-771`), so the gate passes.
The fixer is now barred from the whole call, so this vacuous pass is its only check, and it hits every table-driven test.

### [MEDIUM] Non-test failures have no command to re-run them by

**Location:** `contracts/stencils/gatefix/gatefix-body-card.md:28`

**Issue:** `reportLines` (`internal/gatecli/failurereport.go:236-272`) prints each failed run's label, outcome, an optional way-forward fix and its log, never a command.
The only command the report ever names is `lym loom lint-<name>`, and only in the cut line of a truncated lint step (`failurereport.go:285-286`).
A `build`, `vet tierN`, package-level or uncut lint failure therefore has no "command the report names", and line 29 forbids the whole call.

### [LOW] The re-run form misses nested modules

**Location:** `contracts/stencils/gatefix/gatefix-body-card.md:28`

**Issue:** Explicit targets run in the `-C` directory, else the tier's own directory (`internal/gatecli/tiers.go:278`).
The report names a package by import path, and the red card call ran each nested module in its own directory (`scope.go:246-254`).
The form as written makes the go command refuse a nested module's package from the root.

### [LOW] Long help omits the --gate card exception

**Location:** `internal/gatecli/cli.go:89`

**Issue:** The help line says a narrower scope under the go scoper runs the guard tests outside it.
`scope.go:257` now sets `guardStep: !ownOnly`, and the flag help at `cli.go:125` already says so.

### [LOW] guardStep field comment is stale

**Location:** `internal/gatecli/scope.go:76`

**Issue:** The comment lists `--card` among the forms that plan a guard step, without the card key's exception at `scope.go:245-257`.

### [LOW] Vet step and forced guard step under --gate card are untested

**Location:** `internal/gatecli/scopes_integration_test.go:826`

**Issue:** `_lym/landing/summary.md:16-17` states that `--guards` still forces a guard step and that the vet step stays.
The code does both (`tiers.go:193-206`, `run.go:122`).
No row runs either with `--gate card`, so a regression that ties either one to `guardStep` would pass.

### [LOW] Merge danger leaves out the config meaning change

**Location:** `_lym/landing/summary.md:64`

**Issue:** The rubric makes a two-way call a finding when the diff changes what a config setting means.
`internal/gateslot/template.yaml:54` now gives the `card` key's `scope: card` a different meaning, keyed in code (`scope.go:245`).
The door may well stay two-way: nothing is rewritten, pushed or deleted, and the previous commit `af3813927` narrowed the batch key the same way with no reinterpretation.
But the section does not name the change, and its blast radius `gate` leaves out the implementer and fixer prompts the diff changes.

## Rejected

- Gate-key literal cleanup in `priority.go`, `weaverengine/strand.go`, `runplan/cardgate.go` and `webstercli/beginbatch.go` as unrequested scope (lens:code-scope): once `CardGateKey` is exported, `PATTERN-code-conventions` asks for one named constant per name that can go stale, and the change description names the cleanup (`summary.md:25-26`).
- The guard-step trade figures: 8.1 + 4.4 = 12.5 min per row against 11 batches × 1.2 min / 6 rows = 2.2 min agree with `docs/research/webster/gate-time-2026-11-28.md` (lines 110, 163, 181, 206), and the summary states they are estimates from earlier runs.
  The board entry asks for the call and its why, which the summary gives.
- `PATTERN-config-layers` reinterpretation for the changed `scope: card` meaning: no config value can express the old meaning, and the preceding batch-key narrowing shipped none, so the rule's value-rewrite route does not fit this behaviour change; F7 covers the disclosure.
- Dotnet and none scopers against the spec, `template.yaml`, the `--card` help and the PATTERN bullet: neither scoper plans a guard step (`tiers.go:206-215`, `dotnetscope.go:33`), so the no-guard-step claims hold; the "cards' packages alone" wording describes the go scoper's narrowing, as the dependent-scope wording before it did.
- Weaver batch template lacking the Webster guard-fix bounds: the Weaver template had no such rule for card gate failures before this change either, and its added line tells the implementer that a batch gate failure may name a file no card lists.
- Hub tier lists where the `batch` key omits tier0: the template's `batch` key lists tier0 (`template.yaml:30`), and the plan-level verify keeps running over the whole tree.

The focus entries were each reached: the fixer re-run form (F1, F2, F3), the gatecli wording (F4, F5), the `ownOnly` branch (correct; F6 for its tests), the dotnet and none scopers (rejected above), batch-gate coverage per flow (not a defect), the Webster guard-fix move (consistent with step 2, step 6, the self-fix bound and the status rules), and the summary's figures and Merge danger (figures rejected above, F7).
