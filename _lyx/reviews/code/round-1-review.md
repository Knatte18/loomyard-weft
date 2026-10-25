---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/adoptcli/doc.go:61
    summary: doc.go still calls the dobby start "Last" although the suite start now follows it
    fix: In internal/adoptcli/doc.go line 61, change the opening words Last, after the resumes, to After the resumes, so the dobby-start paragraph no longer claims to be last.
  - id: F2
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/adoptcli/adopt_test.go:505
    summary: The PrimeRecorded half of the suite-start condition is not pinned by any test
    fix: In internal/adoptcli/adopt_test.go, add a TestAdopt row (or an assertion in the DobbiesThenOneSuiteStartAfterTheResumes row) whose request sets only the Prime field and names no slugs, asserting got.Suite is non-nil and w.order ends in the suite entry.
  - id: F3
    severity: LOW
    class: consistency
    origin: lens:code-pattern-fit
    location: internal/adoptcli/deps.go:117
    summary: The suite log path is joined inline in adoptcli instead of through a fabricengine accessor beside HubScratchDir
    fix: Add a fabricengine accessor next to HubScratchDir in internal/fabricengine/junctionnames.go (as for the runproc and failed-push directories) that returns the join of HubScratchDir(hub) and lyxdirs.AdoptDirName, and call it from the StartSuite closure in internal/adoptcli/deps.go.
---

The diff makes the change the amended board entry and the decision record ask for.
The suite starts once per call, after the dobbies, only when an owner was recorded for an entry or the prime.
A refused or ownerless call returns or skips before the seam.
A failed spawn sets `suite_warning` naming `lyx gate test --gate finalize` and leaves every slug outcome and the exit status unchanged.
The hub seam spawns detached in the prime from `ResolveBattenPrime`, truncates the log, logs the spawn and starts nothing under `go test`.
The log path uses only the new `lyxdirs` token under the board-level `.lyx`.

### [LOW] Stale "Last" in the package doc

**Location:** internal/adoptcli/doc.go:61
**Issue:** The dobby-start paragraph opens "Last, after the resumes", but the new `# Step 6` section describes a step that runs after it.

### [LOW] Prime-only owner record is not pinned

**Location:** internal/adoptcli/adopt_test.go:505
**Issue:** The new row passes `Prime: true` with two slugs, so `len(adopted) > 0` already holds. Deleting `|| result.PrimeRecorded` from the condition at adopt.go:229 would fail no test, though the decision record names the prime as an owner-recording case.

### [LOW] Suite log path derived inline

**Location:** internal/adoptcli/deps.go:117
**Issue:** `PATTERN-durable-vs-ephemeral-state` has each module expose a scratch accessor; `runproc` and `failed-push` have one in `fabricengine`. The adopt log joins the path in the `hubDepsAs` closure.

## Rejected

- lens:code-tests F2 (the `suite` and `suite_warning` envelope keys are untested): the package tests the `adopt` function and not the cobra wiring, and the lens could not show that the existing `dobby_warning` field is tested either, so this is the package's convention rather than a gap this diff opens.
- lens:code-pattern-fit finding 2 (step 3 of the long help lost a clause): the change description discloses the trim, and the rationale is still carried by the refusal text and `doc.go`, so no information is lost.
- Handler: truncating the log at each start can garble a still-running earlier suite's output when two adopt calls come close together; the decision record fixes "truncated at each start" and names the two-call case as an accepted bound.
