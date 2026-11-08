---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    origin: lens:code-tests
    location: internal/websterengine/beginbatch.go:562
    summary: BeginBatch passing the last-batch flag to renderBatchGate has no test that fails when the flag is dropped
    fix: In the existing begin-batch prompt test in internal/websterengine/beginbatch_test.go near line 420, assert that the last batch's prompt holds the batch gate line ending in --main-packages and that a non-last batch's prompt lacks it. The recover test at recoverbatch_test.go:1741 is the model.
  - id: F2
    severity: MEDIUM
    class: scope
    origin: lens:code-tests
    location: internal/gatecli/detachfront.go:360
    summary: The detached runner's call words append --main-packages, and no test asserts it, so a capped call and its runner could disagree on the scope unseen
    fix: Add a case to the existing test of testRequest.words, or of the detached runner's command line, in internal/gatecli, that a request with mainPackages set renders --main-packages once and one without renders none.
  - id: F3
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: contracts/stencils/webster/webster-body-implementer.md:106
    summary: The new last-batch exception is not carried into the report-status rules, which still say a batch gate step failing only on failures the batch's cards did not cause does not make the report FAILED
    fix: In the same stencil, at the sentence that says such a batch gate failure does not make the report FAILED and at its repeat in the status rules, add an exception for a failure in the packages --main-packages adds. Reword the line 107 phrase "the one exception to the rule below" so it names both rules.
  - id: F4
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/gatecli/scope.go:283
    summary: withMainPackages' dedupe, append-new-module and no-mains branches, the go list failure refusal, and the none and dotnet scopers adding nothing have no test row
    fix: Add rows beside the "--card with --main-packages" subtest in internal/gatecli/scopes_integration_test.go for a card that already names the main package (listed once), a card in another module, a failing go list (refusal names the module and the way forward), and --main-packages under the none scoper adding nothing.
  - id: F5
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/impactset/mains.go:50
    summary: ModuleMainPackages keeping a package that fails to build, which is the reason for go list -e, and its two directory error branches have no test
    fix: In TestModuleMainPackages add a subtest with a package main holding a type error and assert it is still listed.
---

### [MEDIUM] BeginBatch's last-batch wiring is unpinned

**Location:** internal/websterengine/beginbatch.go:562
**Issue:** The begin-batch prompt test asserts the batch gate line as a substring with no closing backtick, so it passes with or without `--main-packages`. Dropping the flag from `BeginBatch` fails no test, although begin-batch is the main path and the recover path is pinned.

### [MEDIUM] Detached runner's --main-packages words are unpinned

**Location:** internal/gatecli/detachfront.go:360
**Issue:** `words()` appends `--main-packages` so the detached runner repeats the scope of the capped call. Nothing asserts it, so dropping it leaves every test green while the runner runs a narrower scope than the call that was capped.

### [LOW] Stencil status rules do not carry the new exception

**Location:** contracts/stencils/webster/webster-body-implementer.md:106
**Issue:** The added lines make a failure in the added packages the last fork's to fix, but the unchanged rules further down still say a batch gate failure the batch's cards did not cause does not make the report `FAILED`. A reader can take either; the goal of fixing the break in the fork is only reached under the first. Kept at LOW: the added text says "the one exception to the rule below" and reads as controlling.

### [LOW] Untested withMainPackages and refusal branches

**Location:** internal/gatecli/scope.go:283
**Issue:** The one new scope test covers a module the cards already name with a main they do not. The dedupe branch, the append of a module the cards leave out, the empty list, the `go list` failure refusal and the non-go scopers adding nothing, which the decision record states, are all unpinned.

### [LOW] ModuleMainPackages' -e behavior is untested

**Location:** internal/impactset/mains.go:50
**Issue:** `-e` is there so a main package that fails to build stays listed and fails the gate; no test pins it.

## Rejected

- Duplicate `./cmd/lym/...` card target plus derived `./cmd/lym` runs a package twice (lens code-correctness, LOW): costs time only, no wrong result, and card targets in this repository are package paths; not worth a code change.
- Nested modules' mains not covered (lens code-correctness): the decision record says "every package main of the module", so tier0's module is the contract.
- Scope against the decision record: no `gates.batch.last_packages` field, no config step, no Go path in a template, and the plan-level verify is untouched; the diff adds none of these.
- Change description numbers (28 min versus 24 min saved, last batch only): the description states the measured choice and its numbers as the Goal asks; I could not re-derive them because the hub has no `.lym/gate/runs` records in this worktree, so I take them as the writer's claim.
