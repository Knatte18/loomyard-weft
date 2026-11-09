---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/testcover/runner.go:60
    summary: the coverage build runs in the process cwd, not the module root the verb resolved, so a module below the worktree root or a cwd outside it fails every build
    fix: In internal/testcover/runner.go add a Dir field to Toolchain and set cmd.Dir from it in goOutput (or pass -C Dir to go). In internal/gatecli/testmappackage.go set Dir to r.moduleDir in the Toolchain that mapPackage builds. Add a testmap case whose tier dir is a subdirectory of the worktree and whose stand-in go fails unless its cwd holds go.mod.
  - id: F2
    severity: MEDIUM
    class: design
    origin: handler
    location: internal/gatecli/testmappackage.go:139
    summary: a terminating signal during the lym build is recorded as a build failure in the package's map, which then is not retried at the same HEAD; the call context also never reaches coverage builds or tests
    fix: In buildLym, when r.ctx.Err() is non-nil after prebuildLym returns, do not set lymErr to a build failure and leave lymBuilt false; in mapPackage return r.ctx.Err() instead of calling writeFailed in that case. Give testcover.Toolchain a Ctx field (default Background) that command uses as the parent of its timeout context, and set it from r.ctx in mapPackage and runTest, so a signal ends a running build or test through its process group.
  - id: F3
    severity: MEDIUM
    class: scope
    origin: lens:code-tests
    location: internal/gatecli/testmap_integration_test.go:345
    summary: staleness by a changed Go file set or changed file content has no test row, so a broken maps.Equal in stale would pass
    fix: In TestGateTestMap add a subtest after the second call that edits and commits a Go file of a mapped package (for the content case) and adds and commits a new Go file to another (for the set case), calls the verb, and asserts both packages are in built with the new file hashes in their maps.
  - id: F4
    severity: MEDIUM
    class: scope
    origin: lens:code-tests
    location: internal/testcover/runner_test.go:104
    summary: neither the build timeout (BuildBinary and the lym build counted as failed) nor the grace bound with a descendant holding the output pipe has a test
    fix: In internal/testcover/runner_test.go add a TestRunAlone row whose stand-in test starts a child that inherits stdout and sleeps, and assert the timed-out result returns within the grace. Add a BuildBinary case with BuildTimeout set small and a stand-in go that hangs, asserting an error naming the timeout. In TestGateTestMap add a call with redundancy_build_timeout_min shortened through minuteUnit and a stand-in build that hangs for one package, asserting the package is in failed with a timeout message.
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/gatecli/prebuild.go:63
    summary: the lym build has a group kill but no WaitDelay, so a descendant outside the group holding the output pipe stalls it past the build timeout
    fix: In prebuildLym set cmd.WaitDelay to a grace passed in by the caller (testmap passes its request grace; the tier run passes the pipe wait it already uses) before cmd.CombinedOutput.
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/gatecli/testmap.go:374
    summary: a named package with no Go file tracked at HEAD is dropped silently, though D5 requires each skipped package to be listed with its cause
    fix: In mapPackages, when named is true and len(files) is 0, call r.skip(pkg, "no Go file tracked at HEAD") before continue. Add a row for it to the Refusals-style table or the named-package subtest.
  - id: F7
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/gatecli/testmap_integration_test.go:286
    summary: several added behaviours have no row - the mid-package skip counting against --max, the once-per-call lym build with the PrebuiltLymEnv hand-off, the no-hub, no-mapped-tier and unknown-package refusals, and the modulePackages exclusions
    fix: Add rows to the Refusals table for the outside-any-hub, no-mapped-tier and unknown-package refusals with their way-forward text. Add a call with --max 1 over moved and a later stale package, asserting moved is skipped and the later one counts in unbuilt. Give one fixture tier prebuilt_lym and assert the stand-in test sees the PrebuiltLymEnv variable once for two packages. Add testdata, a dot directory and a nested module to the fixture and assert none is walked.
  - id: F8
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/gatecli/doc.go:921
    summary: the doc pins the perishable value "ten-second" for the pipe wait
    fix: In the Coverage maps section of internal/gatecli/doc.go replace "the ten-second pipe wait" with "the pipe wait the gate already uses".
  - id: F9
    severity: NIT
    class: consistency
    origin: lens:code-tests
    location: internal/gatecli/testmap_integration_test.go:190
    summary: the hang test waits a real 5 seconds twice because secondUnit is 250ms
    fix: In TestGateTestMap lower secondUnit in the call helper to 50ms and redundancy_test_timeout_sec in the gate.yaml string so the timeout stays about one second, keeping it long enough for the stand-in to start.
---

### [MEDIUM] Coverage build runs outside the module root

**Location:** internal/testcover/runner.go:60
**Issue:** `Toolchain` has no working directory, so `go test -c <importPath>` runs in the cwd of `lym`. `prepare` resolves the module root from the lowest tier's dir and supports one below the worktree root, yet the build never runs there. The stand-in `go` in the test ignores its cwd, which hides it.

### [MEDIUM] A signal is recorded as a build failure

**Location:** internal/gatecli/testmappackage.go:139
**Issue:** After a signal cancels the call context, `prebuildLym` fails with a cancellation, `buildLym` keeps it as `lymErr`, and `mapPackage` writes a failure map. A failure map is retried only after a HEAD move, so the package stays unmapped at this HEAD for a cause outside it. Coverage builds and tests use `context.Background()`, so a signal does not stop them either.

### [MEDIUM] Staleness by file set or content is untested

**Location:** internal/gatecli/testmap_integration_test.go:345
**Issue:** Only the missing-map, age and failed-build-at-another-HEAD causes have a row. The first acceptance criterion names the file set and content causes, and no test fails if `maps.Equal(m.Files, files)` is broken.

### [MEDIUM] Build timeout and grace bound are untested

**Location:** internal/testcover/runner_test.go:104
**Issue:** The hang stand-in spawns no child, so the group kill and the grace bound are never needed to pass. No test reaches the `BuildTimeout` error of `goOutput` or the timeout branch of `buildLym`.

### [LOW] The lym build has no wait bound after a kill

**Location:** internal/gatecli/prebuild.go:63
**Issue:** `CombinedOutput` after a group kill still waits for the output pipe; a process that left the group and holds it stalls the call past `redundancy_build_timeout_min`.

### [LOW] A named package with no tracked Go file is dropped silently

**Location:** internal/gatecli/testmap.go:374
**Issue:** D5 lists each skipped package with its cause; this package appears nowhere in the output.

### [LOW] Added behaviours without a test row

**Location:** internal/gatecli/testmap_integration_test.go:286
**Issue:** See the fix for the list of behaviours the table and the call sequence do not reach.

### [LOW] Perishable value in the gatecli doc

**Location:** internal/gatecli/doc.go:921
**Issue:** "ten-second pipe wait" names the value of a parameter that is set elsewhere and can change.

### [NIT] Real five-second wait in the hang test

**Location:** internal/gatecli/testmap_integration_test.go:190
**Issue:** The timeout is injectable, but the chosen unit makes two calls wait it out for five seconds each.

## Rejected

- Exit record written on error and refusal-after-prepare paths with a partial count (code-correctness F4): the decision says the verb writes the record as it exits and the focus asks for every exit path, so the code matches; the partial count is the count at that exit.
- Race between the stale check and the lock acquisition (handler): the rebuild after a concurrent finish is wasted work only, since the post-test hash check guards the written map.
- The `--max` default of 5 has no source in the decision record (handler): the verb needs some default and the record does not forbid one.
- The assist reviewer's report: not available; its sandbox fails on this machine, and the orch told this round to consolidate from the three lens forks alone.
