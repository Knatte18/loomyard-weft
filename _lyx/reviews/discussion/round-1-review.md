---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md D5
    summary: D5 offers a package-level var as a time-seam shape, but a test that sets a package var is process-global and cannot run in parallel, defeating the speed goal and PATTERN-test-parallel
  - id: F2
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md D6
    summary: D6 retags tmux-starting integration and smoke files out of every gate that runs them today, with no bound in the Decision; a test may end up under a tag no gate runs
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D6 Rationale
    summary: D6 dates the llm name decision 2026-10-06, a day after the session date and after the dates the record uses; the decision was made 2026-10-05
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D2
    summary: Per-test coverage cannot see work done in a spawned lyx binary, so tests driving the built binary look redundant or fall into no-coverage by accident; D2 does not say how they are classified
  - id: F5
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md D9
    summary: D9 production socket removal checks that no server answers and then removes, leaving a window for a server restarting on the same key; the Decision states neither the residual window nor a guard against it, and no criterion covers it
  - id: F6
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md D7
    summary: D7 says the llmkit os/exec exemption is bounded to LookPath, but PATTERN-testkit's import-level enforcement cannot see which os/exec identifiers a kit uses; the Decision names no mechanism that stops the exemption widening
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D8
    summary: D8 says the five retired IDs' background files are deleted, but only three of them have one (tier-purity, hermetic-git-tests, tmux-test-isolation); time-seam and parallel have PATTERN.md lines only
  - id: F8
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md Constraints and Open risks
    summary: The first Constraints bullet restates D8 and the LLM-guard Open risk restates D7's Bound; the smoke and card-gate risks carry requirements that belong in D6
---

### [BLOCKING] Time seam as a package-level var

**Location:** `decision-record.md` D5, first bullet.
**Issue:**
D5 lets each production wait become "a package-level `var` or an injected field", citing the `ghAuthTokenTimeout` fix as the shape.
That fix is a package var (`internal/githubclient/token.go`) that tests overwrite.
A test that writes a package var is touching process-global state, so under `PATTERN-test-parallel` it must stay serial; the seams then cannot support the parallelism the task sets out to add.
`PATTERN-test-time-seam` also requires the interval or clock to be injectable by the test, which a shared var is not.
The parent review made the same finding.
**Fix:**
Make the shape an injected field or option only, drop the package-var alternative and the `ghAuthTokenTimeout` reference, and state that a seam a test shortens must be per-instance so the test can call `t.Parallel`.

### [BLOCKING] Retag drops tests out of every gate

**Location:** `decision-record.md` D6, with Open risks bullets 4 and 5.
**Issue:**
D6 moves tmux-starting `integration` files to `tmux` and retires `smoke`.
That removes them from `cardGateCommand`'s `-tags integration` run and from plan verifies that run `go vet -tags smoke` or `-tags integration`, a guard downgraded by this Decision.
The Decision does not say what slips past or what bounds it; the only statement is under Open risks, and it ends at "until note `test-tiers` decides", which leaves the tests under a tag no gate runs.
**Fix:**
Add a Bound to D6: until `test-tiers` decides gate selection, the plan verify and every card verify that touches a retagged test run `-tags tmux` wherever those tests ran under `integration` or `smoke` before, and no test may end up under a tag no gate runs.
Add a matching acceptance criterion, and reduce the two Open risks to a pointer or drop them.

### [LOW] Wrong date for the llm name decision

**Location:** `decision-record.md` D6 Rationale.
**Issue:**
"the `llm` name on 2026-10-06" is a day after the session date, and nothing explains a forward reference.
The decision was made 2026-10-05.
**Fix:**
Change the date to 2026-10-05.

### [MEDIUM] Coverage is blind to spawned binaries

**Location:** `decision-record.md` D2.
**Issue:**
D2's premise is that a test's covered block set shows what it exercises.
Tests that drive the built `lyx` binary run the production code in a child process, which an in-process `-cover` test binary does not record.
Such a test covers only its helper blocks, so it is either listed as a candidate subset of everything else, which would send a prune task toward deleting the only test of a CLI path, or lands in "no coverage" only by accident.
**Fix:**
State in D2 that a test whose run spawns the built `lyx` binary or another subprocess is listed apart with the "no coverage" tests, never as redundant, unless the tool collects the child's coverage (`GOCOVERDIR`); name which of the two the tool does.

### [LOW] Socket removal race

**Location:** `decision-record.md` D9, Production bullet and Acceptance criteria.
**Issue:**
The bound (own key, path is a socket, no server answers) leaves a check-then-remove window in which a server restarting on the same key could bind and then lose its socket file.
The Decision does not say whether that window is accepted or closed, and the criterion tests only a socket whose server still answers.
**Fix:**
State in D9 that the removal runs only inside the teardown that just killed the key's server, and that a connection that succeeds or a socket whose identity changed between the check and the removal leaves the file alone; add a criterion for the restart-on-the-same-key case, or state the residual window as accepted.

### [MEDIUM] llmkit exemption bound has no enforcement

**Location:** `decision-record.md` D7, and the `PATTERN-testkit` exemption it adds.
**Issue:**
`PATTERN-testkit`'s enforcement test checks imports, so `os/exec` in `llmkit` admits any `exec` call, not only `LookPath`.
The existing exemptions (`lyxbin`, `tmuxkit`) are bounded by review.
A third one with the same enforcement widens the same way, and D7 names no check that holds it.
**Fix:**
Have the D7 guard test also fail when a file under `internal/testkit/llmkit` uses an `exec` identifier other than `LookPath`, and say so in the Decision.

### [LOW] Background-file count disagrees with the repo

**Location:** `decision-record.md` D8.
**Issue:**
"the five retired IDs' background files are deleted" presumes five files.
`pattern/` holds backgrounds for `PATTERN-test-tier-purity`, `PATTERN-hermetic-git-tests` and `PATTERN-tmux-test-isolation` only.
**Fix:**
Write "the retired IDs' background files are deleted" without the count, and note that `PATTERN-test-time-seam` and `PATTERN-test-parallel` had none.

### [NIT] Restatement in Constraints and Open risks

**Location:** `decision-record.md` Constraints bullet 1; Open risks bullets 4, 5 and 6.
**Issue:**
Constraints bullet 1 restates D8's merge.
Open risk 6 restates D7's Bound.
Open risks 4 and 5 carry requirements that belong in D6 (see the BLOCKING finding on D6).
**Fix:**
Shorten Constraints bullet 1 to the rule the speed fixes must follow, drop Open risk 6, and fold risks 4 and 5 into D6's Bound.
