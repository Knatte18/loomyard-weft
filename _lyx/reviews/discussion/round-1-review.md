---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md (D2)
    summary: D2 states no bound for a keep directive that is detached from its test, duplicated, stale or left on a test that is no longer a candidate, and does not say whether a kept test stays in the covering pool
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md (D3)
    summary: D3 lets a narrowed run splice sections into a report whose header names a tag set, and lets a package section be dropped, with no bound on a different tag set or a package that fails to build
  - id: F3
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md (D1)
    summary: D1 bound does not cover a test that installs a git hook, alias or helper which makes git run module code out of process
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md (Acceptance criteria, third bullet)
    summary: Third acceptance bullet says every untagged test is judged rather than listed as no coverage, but D1 only retires the spawns and unresolved reasons, so skipped, failed and no-block tests stay listed
  - id: F5
    severity: LOW
    class: scope
    location: _lyx/discussion/decision-record.md (Acceptance criteria)
    summary: No scenario for the D2 misplaced or duplicate directive, the D3 section removal, header preservation and missing out file, or the D1 boundary cases of os.Executable, os.Args and the go toolchain and a tagged unresolved call
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md (D1 and D7 Rationale)
    summary: Measured tallies sit in the Rationale of D1 and D7, and the D1 figure of 483 fabricengine tests outside the report disagrees with the support log figure of 602
  - id: F7
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md (D6)
    summary: D6 says a package drops its tmuxkit.Main wiring only if the pattern no longer requires it, which leaves open whether the drop is required, and which condition makes it allowed
  - id: F8
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md (D4 and first acceptance bullet)
    summary: D4 stops when no unhandled candidate remains while the first acceptance bullet stops when no candidate is outside the Kept list, two wordings of one condition
  - id: F9
    severity: NIT
    class: scope
    location: _lyx/discussion/decision-record.md (Acceptance criteria, last bullet)
    summary: The docs bullet names the pattern background and running-tests.md but not the tool's own package doc, which describes the redundancy mode and its exclusion rule
---

### [MEDIUM] D2 keep directive has no stated bound for misplacement, duplication or staleness

**Location:** `_lyx/discussion/decision-record.md`, D2.
**Issue:** D2 removes a guard: a candidate leaves the candidate list on a reason string, which is what the first acceptance bullet then counts as handled.
The Bound line says the directive changes report output only and that the reason is printed, which covers visibility.
It does not say what the tool does with a directive that is separated from its `func Test…` line (D2 says "directly above"), appears twice on one test, or sits on a test that is not a candidate, nor whether a kept test stays in the covering pool that the removable set is computed against.
Without that, a directive left behind by a rename, a fold or a blank line either silently keeps a test out of the list or silently does nothing, and the plan writer must invent the rule.
**Fix:** Add to D2's Bound: a directive counts only when it is the comment line directly above the test function; a second directive on one test, or a directive with no following test function, is an error naming file and line, like the empty reason; a kept test stays in the covering pool and the removable computation.
A directive on a test that is not a candidate is listed under "Kept" with its reason, so it stays visible.

### [MEDIUM] D3 splice has no stated bound on tag set or on dropping a section

**Location:** `_lyx/discussion/decision-record.md`, D3.
**Issue:** D3 leaves "every other section and the header in place" and removes "a package section that no longer has tests".
The header names the tag set the whole report was generated under (`docs/benchmarks/test-redundancy.md` says `under the tags integration,tmux`).
A narrowed run with a different `-tags` splices sections measured under another tag set into a file whose header claims one, and D3 is silent on it.
"No longer has tests" is also undecided between a package whose test files are gone and one whose listing or build failed in this run; the second would silently delete a section.
**Fix:** Add to D3: a narrowed run splices only when its tag set equals the one in the existing header, otherwise it exits non-zero and writes nothing.
A section is removed only when the package lists no tests, and a package that fails to list or build keeps its old section and makes the run exit non-zero.
A missing `-out` file is written whole.

### [LOW] D1 bound misses a test that makes git run module code

**Location:** `_lyx/discussion/decision-record.md`, D1.
**Issue:** D1's premise is that git's own execution is not module code.
A test can make it so: fabricengine installs a post-checkout hook script and tests install pre-commit hooks in the repo they run git in, and a test could install a hook, alias or helper that runs `lyx` or another module command.
The static scan cannot see that, so such a test stops being excluded and its child-process coverage is under-reported.
D1's Bound (candidates are only added, D4 checks each) is sound, but D4 as written reads assertions only, so nothing in the record tells the reviewer to ask this question.
The hook fabricengine installs today only prints a warning, so no current test is known to be affected.
**Fix:** Add one sentence to D1's Bound: the scan does not detect a hook, alias or helper a test installs, and D4's hand check includes whether the candidate makes git run module code, in which case it is kept under D2.

### [LOW] Third acceptance bullet overstates what D1 changes

**Location:** `_lyx/discussion/decision-record.md`, Acceptance criteria.
**Issue:** The bullet requires every test in an untagged file to be judged "rather than listed as 'no coverage'".
The tool also lists a test as no coverage when it was skipped, failed alone or covers no blocks (`noCoverageReason` in `cmd/testtiming/redundancy.go`), and D1 changes none of those.
The bullet contradicts D1, which retires only the spawn and unresolved reasons for untagged files.
**Fix:** Reword to: no test in an untagged file is listed with the spawns or unclassifiable reason.

### [LOW] Acceptance criteria lack scenarios for the boundary cases of D1-D3

**Location:** `_lyx/discussion/decision-record.md`, Acceptance criteria.
**Issue:** The keep directive's empty-reason error and the byte-identical other sections each have a bullet.
Missing scenarios, none of them covered elsewhere in the record:
- D1 names `os.Executable`, `os.Args[0]` and the `go` toolchain as exclusion triggers, but the bullet names only `lyxbin` and tmux files; and "a tagged file with an unresolved call stays excluded" has no bullet.
- D2 has no scenario for a misplaced or duplicate directive (see the D2 finding).
- D3 has no scenario for section removal, the header staying in place, a missing `-out` file or a mismatched tag set.
**Fix:** Extend the third and fourth bullets, or add one bullet each, naming those behaviors as scenarios; one table row each in the tool's existing tests may meet them.

### [LOW] Rationale carries tallies, and one disagrees with the support log

**Location:** `_lyx/discussion/decision-record.md`, D1 and D7 Rationale.
**Issue:** D1's Rationale says the current rule leaves 483 of fabricengine's 697 tests and all 212 shedadapters tests outside the report.
The committed report lists 602 fabricengine tests under no coverage (483 spawns plus 119 unclassifiable), which is the figure the support log gives; 483 is the spawn subset only.
D7 carries "nine in loomcli" and the Open risks name fabricengine and websterengine as the packages D1 grows.
These are deliberation evidence, not requirements: D1's and D7's Decisions stand without them, so the completeness-before-leanness test finds nothing the plan writer needs in them.
The support log already holds the correct per-package tallies.
**Fix:** Delete the tally sentence from D1's Rationale and "(nine in loomcli)" from D7's Decision; the support log's Interview item 2 and 6 keep the numbers.

### [LOW] D6 wiring condition is permissive, not decidable as an action

**Location:** `_lyx/discussion/decision-record.md`, D6 Rationale and Bound.
**Issue:** "A package left with no tagged test file drops its `tmuxkit.Main` wiring only if `PATTERN-test-isolation` no longer requires it" states a necessary condition, not an instruction.
The pattern keys on whether any test file carries an `integration`, `tmux` or `llm` tag, so the condition is decidable from the record plus the pattern, but the record does not say whether the drop is required, or that an `llm` file keeps the wiring.
The seam bound itself (field or option, never a package variable, production passes today's value) is stated and adequate; this finding is only about the wiring sentence.
**Fix:** Replace with: a package whose last `integration`, `tmux` or `llm` test file is retagged untagged may remove its `TestMain`'s `tmuxkit.Main` call, and a package that keeps any such file keeps the call.
Keep `HermeticGitEnv` wherever the package still spawns git.

### [NIT] Two wordings of the D4 stop condition

**Location:** `_lyx/discussion/decision-record.md`, D4 and Acceptance criteria.
**Issue:** D4 repeats until "no unhandled candidate remains"; the first acceptance bullet requires "no candidate outside the Kept list".
After a rerun a deleted or folded test is no longer a candidate, so the two are the same condition, but "unhandled" is undefined.
**Fix:** In D4 say "until no candidate outside the Kept list remains".

### [NIT] Docs bullet omits the tool's own package doc

**Location:** `_lyx/discussion/decision-record.md`, last acceptance bullet.
**Issue:** `cmd/testtiming/main.go`'s header describes the `-redundancy` mode, including which tests it cannot judge (spawning a subprocess), and D1 and D2 change both.
The project rule is that a module's doc lands in the same commit as its behavior change.
**Fix:** Add the tool's package doc to that bullet.

## Checked and not a defect

- The seam bound in D6 (the production seam) states what bounds it; only the wiring sentence needed a fix.
- Ownership of the tool change: the record's Auto-mode assumption, Scope and Open risks, and the support log's Q1, Q8 and Rejected alternatives all say this task owns it and that the hub compares the two PRs; the two files do not disagree.
- The Kept list, the failed-alone rule (D7 and the first acceptance bullet) and the Scope exclusion of speed work agree; seams and fakes are in scope through D6.
- Missing rejected alternatives, the plan-writer notes and call-site enumeration were not flagged, per the rubric.
