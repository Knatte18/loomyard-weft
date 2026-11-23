---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    origin: lens:code-affected-set
    location: contracts/recipes/darn-recipe.yaml:137
    summary: The shipped darn recipe's red-first gate comment still says a check that cannot judge the tests holds the run until a new commit or the record lands, which the diff makes false.
    fix: "In contracts/recipes/darn-recipe.yaml, rewrite the red-first comment lines 137-138 to match internal/shedrecipe/doc.go: say that a hub whose scoper is not go passes the gate, since the check cannot judge the tests there, and that a diff that adds no test holds the run without spending an attempt until a new commit or that record lands."
  - id: F2
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: docs/overview.md:1113
    summary: The overview still says a check that cannot judge the tests holds the handoff, and after the diff only a diff with no test holds.
    fix: "In docs/overview.md line 1113, replace \"a diff with no test, or a check that cannot judge the tests, holds the handoff\" with \"a diff with no test holds the handoff\", and add that a hub whose scoper is not go passes the gate."
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/loomshed/redfirstgate_test.go#TestRedFirstGate_VerdictPerLabelRecordAndExitCode
    summary: No test covers exit 5 together with a runner crash, so deleting the !res.RunnerCrash conjunct in verify's new pass branch (redfirstgate.go:216) breaks no test.
    fix: "Add a row to TestRedFirstGate_VerdictPerLabelRecordAndExitCode after the exit-5 timeout row: its command writes the gate's runner-crash line (the line gateslot.FindRunnerCrash detects) to a temp file, cats it and runs exit 5. Set wantTerminal true, wantRan true and wantFindings to the crash text, \"The red-first check's test runner crashed\"."
  - id: F4
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/landing/summary.md:11
    summary: "The summary claims a diff that adds or changes no test still holds the run as before. On a non-go hub the verb exits 5 before it looks for tests, so such a diff now passes there."
    fix: "In _lym/landing/summary.md, change the sentence \"a diff that adds or changes no test still holds the run as before\" to say this holds on a hub whose scoper is go. Change the Merge danger blast radius from \"loomshed\" to name the gatecli verb message and the darn writer stencil too."
---

The diff does what the board entry asks.
In a hub whose scoper is not go, the red-first gate now skips the check instead of holding for a manual `No test seam` waiver.
`verify` passes `gateslot.RedFirstUnjudgedExit` when the run neither timed out nor crashed, and the arrival and final paths both take that verdict.
Within the verb, exit 5 is reached only from the non-go branch of `runRedFirst`, so no other failure passes by mistake.
The `Root-Cause:` trailer names the defect the diff removes: `holdsForCommitOrRecord` counted exit 5 as a hold.
The help text, the `runRedFirst` doc comment and `gatecli/doc.go` agree that exit 5 means "could not judge", and that a gate passes it.
The stencil drops the waiver ask for a non-go hub and keeps every other rule.

### [MEDIUM] Shipped darn recipe comment still describes the exit-5 hold

**Location:** contracts/recipes/darn-recipe.yaml:137

**Issue:** The red-first entry's comment says "A diff that adds no test, and a check that cannot judge the tests, hold the run without spending an attempt until a new commit or that record lands."
The diff changes the same sentence in `internal/shedrecipe/doc.go` but leaves the recipe's own copy wrong.

### [LOW] docs/overview.md still says an unjudged check holds the handoff

**Location:** docs/overview.md:1113

**Issue:** The Darn-Write paragraph still lists "a check that cannot judge the tests" as holding the handoff.
No repository rule requires this doc to land in the same diff, so the finding is LOW.

### [LOW] Exit 5 with a runner crash is untested

**Location:** internal/loomshed/redfirstgate_test.go#TestRedFirstGate_VerdictPerLabelRecordAndExitCode

**Issue:** The new pass branch excludes `TimedOut` and `RunnerCrash`.
The timeout conjunct has a row at line 160; the crash conjunct has none.

### [LOW] Summary overstates the no-test hold and understates the blast radius

**Location:** _lym/landing/summary.md:11

**Issue:** On a non-go hub the verb returns exit 5 before it discovers tests (`internal/gatecli/redfirst.go:128-131`), so a diff with no test there passes rather than holds.
The blast radius names only loomshed, but the diff also changes the verb's message and the darn writer's instructions.

## Rejected

- Focus question on the hold test's switch from exit 5 to exit 4: not a defect. The test's name and doc describe a refusal hold, and exit 4 still holds.
- Focus question on exit 5 in a resumed hold: not a defect. The hold state lives in memory and only exit 4 sets it now, so `final` on a held tree returns exit-4 findings only.
- Focus question on whether a bug fix with exit 5 passes when the diff adds no test: it does on a non-go hub, which the board entry's "skips red-first" asks for. The summary wording is covered by F4.
- `docs/overview.md:997`: still correct, since exit 5 remains its own exit code.
