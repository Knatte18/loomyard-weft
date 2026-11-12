---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/gatecli/redundant_integration_test.go:92
    summary: The verb's refusals for an unreadable or out-of-range map and for a failed keep scan have no test, though the commit message and doc.go promise them.
    fix: Add two rows to the refusals table in TestGateRedundant. One writes a map whose test names a block index outside its block table, wanting "is unreadable". The other writes a *_test.go file in the package holding a //testtiming:keep line with no reason, wanting "the keep scan of". Each also wants "way forward".
    origin: lens:code-tests
  - id: F2
    severity: LOW
    class: design
    location: internal/gatecli/redundant.go:99
    summary: A map whose Failure field records a later failed build is judged like a current one and the answer does not say so, so candidates may come from a commit the upkeep pass has failed past.
    fix: Add a map_build_failure key to the answer holding m.Failure.Commit and m.Failure.Error when m.Failure is non-nil, omitted otherwise, document it in the gatecli doc.go paragraph for the verb, and extend the verb's test with a map carrying a Failure.
    origin: lens:code-correctness
  - id: F3
    severity: LOW
    class: consistency
    location: docs/research/README.md:132
    summary: The index of speed reports does not list the new coverage-map pilot report.
    fix: Add a line after the testmap-upkeep-2026-11-11 entry linking speed/coverage-map-pilot-2026-11-12/report.md, with a one-line description of the internal/adoptcli pilot and its verdict question.
    origin: lens:code-affected-set
  - id: F4
    severity: LOW
    class: consistency
    location: internal/testcover/verdict.go:121
    summary: The JudgeMap doc comment is stated in terms of another symbol ("as Judge judges") instead of standing alone.
    fix: Reword the second line of the JudgeMap doc comment to state the rule itself, such as "It judges by the blocks of the package's own files, each entry's seconds ordering the greedy pass, and counts only judged entries as covers", with no reference to Judge.
    origin: handler
  - id: F5
    severity: NIT
    class: design
    location: internal/gatecli/redundant_integration_test.go:78
    summary: The one candidate row is tier 0, the zero value, so a wrong or missing Tier mapping in the verb passes.
    fix: Give TestCovSub Tier 1 in the written map and in the want candidate.
    origin: lens:code-tests
  - id: F6
    severity: NIT
    class: consistency
    location: internal/gatecli/redundant.go:89
    summary: The way forward for any redundancyModule error says to set the dir of tier0 or tier1, which is the wrong step when the hub declares neither tier.
    fix: Word the way forward as "declare tier0 or tier1 with its dir at the Go module's root with lym config gate from the prime, then re-run the same command".
    origin: lens:code-correctness
  - id: F7
    severity: NIT
    class: consistency
    location: docs/research/speed/coverage-map-pilot-2026-11-12/report.md:42
    summary: The build table gives 34 entries at line 37 and the verb's own count of tests as 35 at line 42, with no word on the difference.
    fix: Add a clause after the verb's own counts row saying why tests is 35 against 34 map entries, or drop the tests figure from the row.
    origin: handler
---

The diff makes the change the amended Goal asks for.
It adds the one verb the candidate list lacked, `lym gate redundant`, and `testcover.JudgeMap` behind it, and it writes the pilot report on `internal/adoptcli`.
`JudgeMap` leaves keep-marked, outcome and spawn-marked entries out of both the candidates and the covers, as the Goal says, and the verb refuses a missing map, a build-failure-only map and a hub-less call with a way forward.
It builds no other package's map, cuts or folds no test, and leaves the map format and upkeep rules alone.
I rechecked the report's figures against `packages.tsv` and the commit stats, and they hold.
They are 201 packages and 5,418 tests, a median of 8 tests, a maximum of 578, a weighted 80 linked packages per test, 28 commits since 2026-10-20, and the arithmetic of both ends of the estimate.
Two candidates read and two commits compared stay inside the cap.
The adoptcli files that changed between the two commits and the map's commit are `fixloop.go` alone, as the report says.
No PATTERN entry is declared changed, and the added coverage-map bullet in `PATTERN-test-speed.md` is descriptive.

### [MEDIUM] Two promised refusals of the verb have no test

**Location:** internal/gatecli/redundant_integration_test.go:92
**Issue:** The commit message and `doc.go` list refusals for an unreadable map and for a failed keep scan.
`redundant.go` carries both branches, the `checkMapBlocks` path and the `ScanKeeps` path, yet the refusals table holds only the no-hub, no-map and failed-build rows.
A change that dropped either branch, or let a bad block index reach `JudgeMap`, which would panic, fails no test.

### [LOW] A map with a later failed build is judged as current without a note

**Location:** internal/gatecli/redundant.go:99
**Issue:** `Map.Failure` records a build that failed past the map's commit while the entries stay.
The verb prints `map_commit` but nothing marks the newer failure, so a reader takes the candidates as current.

### [LOW] The research index omits the new report

**Location:** docs/research/README.md:132
**Issue:** The index lists every report under `speed/` and the diff leaves it untouched, so the pilot report is unlisted.

### [LOW] The JudgeMap doc comment leans on another symbol

**Location:** internal/testcover/verdict.go:121
**Issue:** "It judges as Judge judges" makes the comment correct only for a reader who knows `Judge`, which the comment convention rules out.

### [NIT] The only candidate row sits at the zero tier

**Location:** internal/gatecli/redundant_integration_test.go:78
**Issue:** `TestCovSub` is tier 0, the zero value of `Tier`, so a verb that dropped the mapping would still pass.

### [NIT] One way forward fits only one cause of the module error

**Location:** internal/gatecli/redundant.go:89
**Issue:** `redundancyModule` also fails when the hub declares neither tier0 nor tier1, where setting a dir is the wrong step.

### [NIT] The build table's test count differs from its entry count

**Location:** docs/research/speed/coverage-map-pilot-2026-11-12/report.md:42
**Issue:** The result row says 34 entries and the verb's counts row says 35 tests, and the report does not explain the gap.

## Rejected

- A `..` in the package argument reads a `map.json` outside the maps directory (code-correctness F3): an operator-only read-only argument, and the read succeeds only for a file that is a map, so nothing is exposed.
- A redundancy finding against `TestGateRedundant` for repeating `JudgeMap`'s containment case: the report lists it as not judged, and it pins the verb's output shape, the keep reasons and the refusals, which the tier0 test does not see.
