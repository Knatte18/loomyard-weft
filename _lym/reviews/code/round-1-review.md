---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: contracts/stencils/darn/darn-fragment-make-change-gated.md:24
    summary: The new measuring route names a tier and a target list instead of a gate key, against PATTERN-test-speed's rule that a stencil names a gate key, never a tier list or target list.
    fix: In step 3 of contracts/stencils/darn/darn-fragment-make-change-gated.md, replace the call `lym gate test --tier <n> <its package or test project> -- <args>` with the darn-write gate key's call plus the touched target and the class-selecting args, `lym gate test --gate darn-write <its package or test project> -- <args>`. That is the key the row's verify gate runs (internal/loomcli/gatefixer.go darnWriteGateKey), and the rule's own exception lets darn add touched targets to a key's call. Drop the --tier wording and keep the one-class, run-once limit.
    origin: assist:assist
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/gatecli/run.go:83
    summary: In a hub that declares no tier, a --gate call exits 0 before the key's lint steps run, so the template card key's comments and pattern lints are skipped silently.
    fix: In runTest in internal/gatecli/run.go, keep the tierless notice. Then, instead of returning 0, set request.root with buildRoot(location, dir) and run runLints(request.root, cfg.Gates[request.gate].Lints). Print each failed lint as the normal path does (printFailedRuns), and exit 1 when any lint fails, else 0. Add a row beside the tierless case in internal/gatecli/test_integration_test.go where the key carries a lint that fails, asserting exit 1 and that no tier ran. Update the gatecli doc.go sentence to say the call runs the key's lints and no tier.
    origin: lens:code-correctness
  - id: F3
    severity: MEDIUM
    class: consistency
    location: contracts/stencils/darn/darn-fragment-make-change-gated.md:26
    summary: The new text forbids a raw test run and has the writer name a no-test project to its parent, while the gate's own no-test line tells the same reader to run those tests by hand with dotnet test and an MSBuild property.
    fix: In the fragment's step 3, replace the last new sentence with one that sends the rerun through the same gate call. When the gate's report says a project ran no test, re-run the same gate call once with the property the report's line names among the args after --. Never run the raw command the report suggests. When that run still runs no test, name the tests to the parent.
    origin: lens:code-affected-set
  - id: F4
    severity: LOW
    class: design
    location: internal/gatecli/scope.go:337
    summary: The two new fall-back-to-whole branches of allScope, a worktree with no parent branch and a merge base that cannot be read, have no test row.
    fix: In TestGateTest_Scopes in internal/gatecli/scopes_integration_test.go, add rows beside the new inert-only finalize row. One row runs `--gate finalize` from prime after writeInert and an inert-only edit, and asserts that the tier runs whole and the scope-is-empty line is absent. A second row runs in a pair whose origin record names a parent branch that no longer exists, and asserts the whole run and the stays-whole stderr line.
    origin: lens:code-tests
  - id: F5
    severity: LOW
    class: consistency
    location: internal/gatecli/cli.go:73
    summary: The lym gate test help still says a key of scope all runs every tier whole, but an inert-only diff now gives it the empty scope.
    fix: In the testCmd Long text in internal/gatecli/cli.go, change the scope-all clause so it runs every tier whole unless every file changed since the merge base with the parent branch is inert, which runs no tier. Leave explicit --all whole.
    origin: lens:code-affected-set
  - id: F6
    severity: LOW
    class: consistency
    location: internal/gateslot/template.yaml:21
    summary: The gate template's comments still call scope all the whole tree, and say a tier a key names must be declared when the gate runs, which no longer holds for the template's own tiers {} state.
    fix: In internal/gateslot/template.yaml, change the scope-all description on line 21 to note that an inert-only change runs no tier. Change the line-28 comment to say that, while tiers is empty, every key runs with nothing to run, and that once any tier is declared a key's tiers must be declared.
    origin: lens:code-affected-set
  - id: F7
    severity: LOW
    class: consistency
    location: contracts/stencils/darn/darn-fragment-make-change-gated.md:1
    summary: The fragment's header comment and darnengine's docs still say the gated writer runs no gate, though step 3 now names a one-class measuring exception.
    fix: Update the header comment of contracts/stencils/darn/darn-fragment-make-change-gated.md (lines 1-2), the package doc at internal/darnengine/doc.go:79, and the GateFixed field comment and DarnTable doc comment in internal/darnengine/spec.go. Each should say the writer and its subagents run no gate except the writer's single measuring run of one test class.
    origin: lens:code-affected-set
  - id: F8
    severity: NIT
    class: consistency
    location: docs/dotnet-hub.md:131
    summary: The doc says Parent-Align is the one planned whole-solution run per task, which no longer holds for an inert-only branch.
    fix: In docs/dotnet-hub.md near line 131, add that an inert-only branch's Parent-Align and Finalize gates run no tier.
    origin: lens:code-affected-set
---

### [BLOCKING] The measuring route names a tier and targets instead of a gate key

**Location:** contracts/stencils/darn/darn-fragment-make-change-gated.md:24
**Issue:** `pattern/PATTERN-test-speed.md:228` says a site or stencil names a gate key, never a tier list, scope form, test command or target list.
Its index entry reads "via `lym gate test` by gate key".
The new exception tells the writer to run `lym gate test --tier <n> <its package or test project> -- <args>`.
The `<n>` also leaves the writer to guess which tier holds the class.
The record declares no PATTERN change, and no commit carries a `Pattern-Departure:` trailer.
A compliant route exists: the rule's own darn exception adds touched targets to a key's call, and explicit targets beside `--gate` replace the key's scope (`internal/gatecli/cli.go:74`).

### [MEDIUM] The tierless early return skips the key's lints

**Location:** internal/gatecli/run.go:83
**Issue:** The template's `card` key carries `lints: [comments, pattern]` (`internal/gateslot/template.yaml:28`), and the template declares `tiers: {}`.
`runLints` reads only the root and the lint names (`run.go:216`), so it needs no tier.
The new return at `run.go:83-86` comes before `request.root` is set and before the lint loop.
A fresh hub's card gate therefore passes a comment-lint or PATTERN-headroom violation that the same key catches once a tier exists.
The message "has nothing to run and passes" is also wrong for such a key.

### [MEDIUM] The stencil's no-test handling contradicts the gate's own way forward

**Location:** contracts/stencils/darn/darn-fragment-make-change-gated.md:26
**Issue:** The fragment says "Never run a raw test command for it", and that when the report says a project ran no test, the writer names the tests to its parent.
The report line the writer then reads (`internal/gatecli/tierrun.go:486`) says to run those tests by hand with `dotnet test <project> -p:<property>=true` when `refuse_raw` allows it.
The writer gets two opposite instructions for the same report.
That is the round-wasting confusion issue #135 describes.
A tier whose command takes `{args}` can carry the property inside the gate call.

### [LOW] allScope's fall-back branches are untested

**Location:** internal/gatecli/scope.go:337
**Issue:** The new row at `scopes_integration_test.go:630` runs in a pair with a readable parent branch.
The prime `--gate finalize` case at `test_integration_test.go:756` has no `inert_paths`, so it returns at `scope.go:334` and never reaches `parentMergeBase`.
No row would fail if the no-parent branch or the unreadable-merge-base branch returned the empty scope.

### [LOW] The help text still says scope all runs every tier whole

**Location:** internal/gatecli/cli.go:73
**Issue:** The help reads "scope all runs every tier whole", but `allScope` (`scope.go:342-346`) now gives such a key the empty scope over an inert-only diff.

### [LOW] The gate template's comments are stale

**Location:** internal/gateslot/template.yaml:21
**Issue:** Line 21 describes scope all as the whole tree.
Line 28's comment says a tier a key names must be declared when the gate runs.
`Config.Gate` now passes every key while `tiers` is empty (`tiers.go:371-373`), which is the template's own state at line 18.
New hubs read these comments.

### [LOW] The darn docs still describe an unconditional no-gate rule

**Location:** contracts/stencils/darn/darn-fragment-make-change-gated.md:1
**Issue:** The fragment's header comment says "the writer and its subagents run no gate".
`internal/darnengine/doc.go:79` and the `GateFixed` and `DarnTable` comments in `internal/darnengine/spec.go:68,77` say the same.
Step 3 now names an exception for the writer.

### [NIT] dotnet-hub doc overstates Parent-Align's whole run

**Location:** docs/dotnet-hub.md:131
**Issue:** "Parent-Align is then the one planned whole-solution run per task" no longer holds for an inert-only branch, whose scope-all gates now run no tier.

## Rejected

- Scope lens F1, card keys not narrowed by inert_paths: a card scope runs the packages the cards name, not a diff. The issue's failing keys, diff and all, are both narrowed now. The doc keeps "A `--card` scope is not narrowed by `inert_paths`" as stated, pre-existing behavior. The fork did not verify its dotnet hypothesis.
- Assistant F3, a `carry: false` tier still runs over an inert-only scope-all gate: `carry: false` is a hub's explicit choice that a tier always runs whole. Every existing inert-only diff scope treats it the same way (`scopes_integration_test.go:250-271`). The issue's hub sets no such key, so its Finalize now runs no tier.
- Assistant F4 at MEDIUM: merged into F4 at LOW. It is a coverage gap on two conservative fall-back branches, not a wrong behavior.
- Issue #135's first ask, that a refused or neutered `dotnet test` say so: lym neither neuters nor silently refuses it. The refuse-raw row denies with a message and a way forward (`internal/shellguard/rows.go:463-472`). The silent exit 0 is a project that sets `IsTestProject` false, which the gate already reports (`tierrun.go:486`). The board's ask is conditional, so no change is owed.
