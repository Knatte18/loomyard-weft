---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: assist:assist
    location: internal/gatecli/mutation.go#mutationStep.run
    summary: The new step adds a branch on the language literal go (s.reach.cfg.Scoper != go), which PATTERN-code-conventions forbids adding outside a language table.
    fix: Do not add a new scoper literal branch in mutationStep.run. Give the scoper a row-keyed capability, such as a table in internal/gateslot keyed by scoper name that says whether the scoper has Go coverage maps, and read the mutation step's eligibility through it. The new mutationStep.run case then asks that table instead of comparing with the go literal. The existing literals in reach.go and redundancy.go may move onto the same lookup in the same change, since the rule says a change that touches debt moves it.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/gatecli/mutation.go#mutationStep.answer
    summary: A report whose mutation check failed or was cancelled still records the changed functions and file hashes, so it matches HEAD and feeds Code-Review zero counts, and the partial findings are lost.
    fix: In mutationStep.answer, when mutationCall.run returns an error, append the error and return the report before filling Functions and FileHashes, so MutationReportMatches can never match it. Add a row to the TestGateMutate report-step subtest that makes the check fail, for example with mutation_budget_min unreachable or a slot refusal, and assert that MutationReportMatches(HEAD) is false and the report's Errors names the failure.
  - id: F3
    severity: MEDIUM
    class: consistency
    origin: lens:code-affected-set
    location: internal/gateslot/doc.go:65
    summary: The gateslot package doc's list of non-tier slot holders and its bound-key paragraphs omit the mutation report step and the even spread of mutation_max_mutants.
    fix: In internal/gateslot/doc.go, add the mutation report step of lym gate test to the non-tier holder list near line 65, saying it waits for its slot after its refresh of the worktree's maps and holds it to its end. In the bounds paragraphs near lines 187-204, name the mutation report step beside lym gate mutate for redundancy_test_timeout_sec, redundancy_test_memory_mb, reach_budget_min, mutation_max_mutants and mutation_budget_min. State that mutation_max_mutants is spread evenly over the changed functions when the cap is short of one each.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/gatecli/mutation.go#mutationStep.run
    summary: The step's judge-nothing branches have no test, though the reach step pins its own; these are red gate, no hub, non-go scoper and no parent branch.
    fix: Add rows to TestGateMutate beside AGateCallListingMutationWritesItsFindingsAndPrintsOneSummaryLine, following the reach step's rows in reach_integration_test.go. Cover a red tier (a tier0 command of false), whose report must carry red_gate and noteRedGate with exit code 1 unchanged. Cover a worktree with no parent branch, whose report must carry noteNoParent. Assert each report's single summary line too.
  - id: F5
    severity: MEDIUM
    class: design
    origin: assist:assist
    location: internal/gatecli/mutation.go#mutationStep.answer
    summary: No test asserts that the step writes its survivors, unjudged mutants, uncovered blocks and packages not judged into mutation.json; the only step row yields one killed mutant and empty lists.
    fix: Extend the TestGateMutate report-step coverage with a diff whose check yields a survivor and an uncovered block, as the verb rows do with Scale and Spare. Read the report with ReadMutationReport and assert report.Survivors lines and report.Uncovered entries match the expected ones.
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/gatecli/mutationreport.go#MutationReportMatches
    summary: MutationReportMatches is tested only for a true match; its refusal of a note, red-gate or prime report and of a tree that changed a recorded function has no row.
    fix: In the TestGateMutate report-step subtest, after the HEAD match, commit a change to Clamp and assert MutationReportMatches at the new HEAD is false. Add a tier0 table test of MutationReportMatches for reports with Note, RedGate and Prime set, each answering false; it returns before any git read.
  - id: F7
    severity: LOW
    class: design
    origin: assist:assist
    location: internal/testcover/mutant.go#logsOnly
    summary: logsOnly marks a whole if statement noise when its body only logs, even when its condition has a side effect, so a changed constant in a call such as counter.Add(1) inside the condition loses its mutant.
    fix: In logsOnly, treat an if statement as log-only only when its condition holds no call expression, checked with ast.Inspect over s.Cond. Add a TestPlanLineMutants row with an if whose condition calls a function with an integer argument and whose body only logs, asserting the argument's line keeps its change-constant mutant.
  - id: F8
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/testcover/linemutant_test.go#TestPlanLineMutants
    summary: The noise rules' else and else-if branches, the init refusal, the reversed GOOS operand order and a shadowed runtime local have no row.
    fix: Extend lineMutantSource with an if-else whose both branches only log, an else-if chain that only logs, an if with an init statement whose body only logs, a comparison written as a string literal == runtime.GOOS naming another platform, and a function whose local named runtime shadows the import. Add TestPlanLineMutants rows asserting the first, second and fourth get no mutant and the third and fifth keep theirs.
  - id: F9
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/loomcli/mutationnote.go#renderMutationReportNote
    summary: The note branch for a report that judged nothing for a reason other than a red gate, and the branch for a HEAD match error, have no row.
    fix: Add two rows to TestRenderMutationReportNote. In one, a found report carries Note noteNoParent text and the want is the judged-nothing line. In the other, matchesHead returns an error and the want is the could-not-be-matched line naming that error.
  - id: F10
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/landing/summary.md:19
    summary: The change description claims every package and file now gets a share under the cap, but the even spread over functions does not guarantee that.
    fix: In _lym/landing/summary.md, replace the sentence claiming every package and file gets a share. Say that the cap's mutants are spread evenly over the changed functions in the order of package, file and name, so the later packages and files are sampled too. In the selectWithinCap doc comment in internal/gatecli/mutatecheck.go, replace judged across its packages and files with sampled evenly across its functions.
  - id: F11
    severity: LOW
    class: design
    origin: assist:assist
    location: internal/gatecli/mutation.go#mutationStep.answer
    summary: The report's match against HEAD hashes only the Go files of the changed packages, so a later change to a testdata file those packages' tests read leaves a stale verdict matching HEAD.
    fix: State the bound in the MutationReport and MutationReportMatches doc comments and in the gatecli doc's Mutation report section. A change to a non-Go file a package's tests read, such as testdata, leaves the report matching HEAD.
---

### [BLOCKING] The step adds a branch on the language literal `go`

**Location:** internal/gatecli/mutation.go#mutationStep.run

**Issue:** `mutationStep.run` adds `case s.reach.cfg.Scoper != "go":`.
`PATTERN-code-conventions`, under "No hard-coded choice", says no code path branches on a language literal outside the language table.
It also says an existing literal is debt that a change never adds to.
The scoper values `go`, `none` and `dotnet` name languages, and this line is a new instance beside the existing ones in `reach.go` and `redundancy.go`.
No commit in the range carries a `Pattern-Departure:` trailer.
A violation of a listed constraint is BLOCKING under this round's rules, however small it is and even though the step really is Go-only.

### [MEDIUM] A failed check writes a report that matches HEAD

**Location:** internal/gatecli/mutation.go#mutationStep.answer

**Issue:** `diffMutation.run` returns `mutateOutcome{}, err` on a slot, build, run or context failure (`internal/gatecli/mutatecheck.go`, `run`), so `found.outcome` stays empty.
`answer` still fills `Functions` and `FileHashes` from `found.funcs` and sets no `Note`.
So `MutationReportMatches` holds for HEAD, and Code-Review reads "matches HEAD's tree" over zero counts and one error.
That contradicts the doc's "never matches a report that judged nothing".
The reach step's error paths record no functions, so they never match.

### [MEDIUM] The gateslot doc omits the new slot holder

**Location:** internal/gateslot/doc.go:65

**Issue:** The non-tier holder list names the redundancy step, the reach refresh, `lym gate redundant` and `lym gate mutate`, but not the mutation report step.
That step takes a slot through `mutationCall.run`'s `mutationCheck` under `mutationSite`.
`PATTERN-gate-slot` allows a non-tier gate build only "in one of the non-tier holders `internal/gateslot`'s doc lists".
The bound paragraphs at lines 187-204 still describe `mutation_max_mutants` and `mutation_budget_min` for `lym gate mutate` alone, with the old spread.
The diff updated the matching `config.go` field comments but not this doc.

### [MEDIUM] The step's judge-nothing branches have no test

**Location:** internal/gatecli/mutation.go#mutationStep.run

**Issue:** The only test of the step is the judged path in `TestGateMutate`.
The red-gate, no-hub and non-go-scoper branches of `run`, and the no-parent branch of `answer`, have no row.
Breaking one would run a full mutation check on a red gate, or write the wrong note, and no test would fail.
`reach_integration_test.go` pins the reach step's counterparts.

### [MEDIUM] Nothing asserts the written findings lists

**Location:** internal/gatecli/mutation.go#mutationStep.answer

**Issue:** The board entry's third point asks for the full list in `mutation.json`, and the marker feeds the survivors to review.
The step's one integration row yields a single killed mutant, so the survivors, unjudged, uncovered and not-judged lists are all empty.
Dropping the line that assigns `report.Survivors`, `report.Unjudged` and `report.Uncovered` would leave every test green.
`TestRenderMutationReportNote` builds its reports by hand.

### [LOW] MutationReportMatches has no false row

**Location:** internal/gatecli/mutationreport.go#MutationReportMatches

**Issue:** The review note relies on a false answer to keep a stale report out of Code-Review.
Only the true case at HEAD is asserted.
The shared `changedFunctionsMatch` is pinned through the reach tests, but the wrapper's own guard on note, red-gate and prime reports is not.

### [LOW] An if with a side-effecting condition counts as log-only

**Location:** internal/testcover/mutant.go#logsOnly

**Issue:** `logsOnly` checks the init statement and the branches of an `if`, never its condition.
For `if counter.Add(1) > 10 { logger.Info(...) }` the whole statement becomes a noise span, so the change-constant mutant of `1` is dropped, though it changes the counter's state.
The board entry asks to skip "a statement that only logs", and this statement does more.

### [LOW] The noise rules lack rows for else, init, reversed operands and shadowing

**Location:** internal/testcover/linemutant_test.go#TestPlanLineMutants

**Issue:** The fixture has no `else`, no `else if`, no `if` with an init statement, no literal on the left of a `runtime.GOOS` comparison, and no local shadowing `runtime`.
Breaking the `default: return logsOnly(branch…)` arm, the `Init` refusal or the operand-swap loop in `comparesOtherPlatform` fails no row.

### [LOW] Two note branches have no row

**Location:** internal/loomcli/mutationnote.go#renderMutationReportNote

**Issue:** `TestRenderMutationReportNote` covers no report, red gate, prime, other tree and match.
It has no row for a found report with a non-red `Note`, such as no parent branch, and none for `matchesHead` returning an error.

### [LOW] The change description overstates the spread

**Location:** _lym/landing/summary.md:19

**Issue:** The summary says that on a diff with more changed functions than the cap, "every package and file now gets a share".
`spreadEvenly` picks indexes `i*len/n` over the sorted functions.
With eleven functions, ten in package `a` and the last in `b`, and a cap of 5, it picks indexes 0, 2, 4, 6 and 8, so `b` gets none.
The `selectWithinCap` doc comment's "judged across its packages and files" leans the same way.
The board entry asks only that a large diff is no longer judged on its first functions, which the code does.

### [LOW] Non-Go test inputs escape the match

**Location:** internal/gatecli/mutation.go#mutationStep.answer

**Issue:** `FileHashes` holds `goFileHashes` of each changed package, the package's `.go` files only.
A covering test that reads a `testdata` file can change its verdict when that file changes, and the report still matches HEAD.
The reach report shares the matcher and the same limit, so this is a bound to state, not a reason to redesign.

## Rejected

- code-scope F1, the `, N errors` suffix on the summary line: the line carries all six counts the board entry lists, and an error count beside them keeps a failed step from printing all zeros. Nothing in the entry rules it out, so no unlicensed change.
- assist F7, the `Loosened-Check:` trailer on `TestLoadConfig`: the removed row asserted the refusal of `mutation`, which the board entry orders to become an accepted report. That is a contract change the answer key asks for, not a widened tolerance, so a measured cause has nothing to measure. The trailer names the row and the reason.

## Review notes

The spread holds its cap and is deterministic: the groups are sorted after map iteration, and the three test rows trace by hand.
The other-platform skip covers only the comparison, and a comparison naming this platform keeps its mutant.
The marker is threaded through the rubric, `RubricNotes`, `BouncerConfig`, `shedrecipe.Env` and `loomcli` wiring, and the note is read only when the report matches HEAD.
Commit 28216d3f's "bulk of the survivors no test could reasonably kill" matches the trial report's "most of the 12".
No test was run in this review; the verify gate covers the build and the suite.
