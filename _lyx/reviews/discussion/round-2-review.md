---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: _lyx/discussion/decision-record.md:Decision 1, Decision 10, Acceptance criteria
    summary: A candidate kept only because a core or testkit file names it has no allowed reason, yet Decision 10 forbids annotating a test with nothing unique to pin and the first acceptance criterion demands a directive on every listed candidate
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:Decision 12, Acceptance criteria
    summary: The coverage rule explains drops by deleted removable candidates while its own rationale says removable deletions leave every block covered, and the figure measured under -coverpkg=./... is not defined per package
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:Decision 10, Decision 2
    summary: A test that received a moved assertion is itself a removable candidate on the next rerun, and nothing stops the repeat loop from deleting it or from annotating it without a reason that fits the rule
  - id: F4
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:Decision 9
    summary: Decision 9 does not say who performs the section splice or how, contradicts Decision 10 on how often the rerun runs, and cannot hold "adds or removes none" for a package that ends with no tests
  - id: F5
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:Decision 5
    summary: The reason an integration test stays because a shared fake is missing is kept only in the PR body, with no durable home beside the test
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:Decision 2, Acceptance criteria
    summary: The per-deletion receiver list is required by Decision 2 but no acceptance criterion checks that the PR lists it
  - id: F7
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md:Decision 8
    summary: The isolated re-run command names no package, and go test needs one
---

### [BLOCKING] A candidate kept for a core reference has no reason that fits Decision 10

**Location:** `decision-record.md` Decision 1 (last two lines), Decision 10, first acceptance criterion.
**Issue:** Decision 1 keeps a test that a core or testkit file names, "not deleted or renamed", even when the report lists it as a removable candidate.
Such a test pins nothing its covering tests do not.
Decision 10 requires a `//testtiming:keep <reason>` on every candidate the rerun still lists, and says the reason "names what the test pins that its covering tests do not"; it also says "a candidate with nothing unique to pin is deleted or folded, never annotated".
The first acceptance criterion needs the directive on every listed candidate.
For this test all three cannot hold: it may not be deleted or renamed (Decision 1), may not be annotated (Decision 10), and must carry a directive (acceptance).
Core's own tool, which I read in `test-suite-prune-core`'s D2, only requires a non-empty reason, so a core-reference reason is accepted there.
**Fix:** Add a reason class to Decision 10: a test a core or testkit file names is kept with a reason naming that file, and Decision 1's bound points at it.
Restate the "never annotated" sentence so it applies to a candidate with no unique assertion and no core reference.

### [MEDIUM] Coverage rule and rationale disagree, and the measured subject is undefined

**Location:** `decision-record.md` Decision 12, "The PR states non-test statement coverage per in-scope package" criterion.
**Issue:** The rule says a drop is named in the PR with "the deleted `Removable: yes` candidates whose blocks the report shows covered elsewhere".
The rationale says "the report's removable set leaves every block covered by construction".
If that holds, a removable deletion never drops a block, so no drop is explained by one, and every drop is a mistake.
`cmd/testtiming/redundancy.go` measures each package's tests under `-coverpkg` over every module package, so a package's tests also cover other packages' code.
Decision 12 uses `-coverpkg=./...` but never says whether the per-package figure is the package's own files, or the whole profile.
`go test` prints one total for the whole `-coverpkg` set, which is not "non-test code in this package".
**Fix:** Define the figure: the covered fraction of statements in the package's own non-test files, taken from the profile of that package's own tests under `-coverpkg=./...`, filtered by directory.
Make every drop a mistake that is restored by bringing back or extending a test, except one traced to a block that flips between two runs of the unchanged tree, which the PR records as noise.
Align the acceptance criterion with this.

### [MEDIUM] A receiving test can be deleted or left without a fitting reason by the repeat loop

**Location:** `decision-record.md` Decision 2 (moved assertion), Decision 10 (repeat loop).
**Issue:** Decision 2 moves a unique assertion into a covering test.
That test's blocks are unchanged, so on the next rerun it can be listed as a candidate, even as `Removable: yes`.
Decision 10 then tells the prune to decide per listed candidate again, and the receiver has nothing unique by block coverage.
Nothing says the receiver survives, which would silently lose the assertion Decision 2 promised to keep.
**Fix:** State that a test that received a moved assertion is never deleted later in the prune; if the rerun lists it, it carries a keep reason naming the moved assertion and its source test.
Add that reason class to Decision 10.

### [MEDIUM] Decision 9 leaves the splice unowned and clashes with Decision 10

**Location:** `decision-record.md` Decision 9 and Decision 10.
**Issue:** `cmd/testtiming` at this base takes one pattern in `-pkg` and writes the `-out` file whole with `os.WriteFile`, so one invocation per package into its own scratch file is achievable, and the stated rerun is sound.
The splice is not performed by the tool; Decision 9 says "this task therefore splices outside the tool" without saying who does it (a card, the PR author, the hub) or how, for a section count in the report that is not small.
Decision 9 says the rerun runs "once per in-scope package", while Decision 10 repeats the rerun until the list is clean; the two do not say the same.
A package whose tests all disappear is skipped by the tool (`timing.noTests`), so its scratch file has no `## <package>` section, and "adds or removes none" cannot hold for it.
**Fix:** Say that intermediate reruns in Decision 10 are scoped to one package at a time and that Decision 9's per-package run is the final one.
Name the final card as the splice performer, one section at a time, and state that a package left with no tests keeps no section.
Put scratch files under `.scratch/`.

### [MEDIUM] The unmade tier move's reason has no durable home

**Location:** `decision-record.md` Decision 5.
**Issue:** When a move needs a new shared fake, "the PR body states that reason and names the missing fake".
The test stays in the `integration` file, and a later pass over the same leads finds it again with no record of why it stayed.
Decision 10 already rests on the premise that a reason only in the PR body is not where later runs look, and it fixed that for kept candidates; this case has the same gap.
**Fix:** Keep the PR-body line as the hand-off to the hub and add a one-line comment above the test naming the missing fake, so the reason lives with the test.

### [LOW] Receiver list has no acceptance check

**Location:** `decision-record.md` Decision 2 bound, acceptance criteria.
**Issue:** Decision 2 says a deletion with unique assertions and no listed receiver "is not allowed", but the acceptance criteria only say the assertions "live on in a covering test", not that the PR lists the receiver.
**Fix:** Extend that acceptance criterion: the PR lists the receiving test for each deleted candidate whose assertion moved.

### [NIT] Isolated re-run command has no package

**Location:** `decision-record.md` Decision 8.
**Issue:** `go test -tags integration,tmux -run '^Name$'` names no package, so it runs the current directory only.
**Fix:** Add `<package>` to the command.

## Regression note

Round 1's F5 asked for the coverage rule that F2 above finds self-contradictory; its own text noted that both could not hold, and the fix tied drops to removable deletions anyway.
F2 settles it the other way: a drop is a mistake.

## Not defects

- Decision 6 states the tag set for the unused-helper rule ("under any tag, `llm` included"), so the focus item on it is met.
- Decision 10's stop is stated (the rerun lists only candidates that carry a directive), and the check by comparison is workable because the tool at this base lists candidates regardless of the directive.
- Decision 1 states the winner against the no-dangling-reference criterion (the test is kept and the reference listed for the hub); F1 concerns only the keep reason.
