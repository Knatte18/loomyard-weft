---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/loomcli/sharedbootstrap.go:185
    summary: A pins file the first seed wrote is never committed when that seed's commit fails or the process dies before it, so it stays untracked and the run's precondition scan fails on it.
    fix: In seedAndCommitBootstrap step 3, append shedrun.StencilPinsRel(c.location, shedrun.SelfRunID) to commitPaths whenever shedrun.StencilPinsFile(c.location, shedrun.SelfRunID) exists (os.Stat succeeds), not only when this invocation wrote it, so a re-entry commits pins an earlier call left uncommitted; keep the write itself gated on the first seed.
  - id: F2
    severity: MEDIUM
    class: consistency
    origin: lens:code-affected-set
    location: internal/loomcli/stencilpins.go:66
    summary: The parent notice and the start and resume help promise that lym stencil diff shows each change since the run started, but diff <name> renders the stamped base against the shipped default and shows nothing after a refresh installed the new default.
    fix: Reword stencilsChangedNotice so it names lym stencil diff as the command to inspect each named stencil against its shipped default and its board copy, without claiming it shows the change since the run started; make the same wording change in the start long help (start.go, the stencils_changed sentence), the resume long help (resume.go, the same sentence) and loomcli/doc.go's notice paragraph.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/loomcli/start.go:628
    summary: No test pins that a successful lym loom start lists stencils_changed and posts one parent notice, though start is the path batten resumes child runs through.
    fix: Add a start test, at the lowest tier its receiver allows, that writes the run's stencil pins, edits one installed stencil, runs start to success with a recording notifyParent, and asserts stencils_changed names the edited stencil and exactly one notice was posted.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/loomcli/stencilpins_integration_test.go:29
    summary: No test pins that the seed commit carries the stencil pins record; the bootstrap test sets FABRIC_SKIP_GIT, so a wrong StencilPinsRel pathspec or a dropped commitPaths append passes every test.
    fix: Pin it in the tier-1 bootstrap test with a git-backed pair (no FABRIC_SKIP_GIT) by asserting the pins file is tracked after the first seed, which also covers F1's re-entry commit; or, if that is too heavy, assert in shedrun.TestStencilPins_RoundTrip that filepath.Join(l.AnchorPath(), StencilPinsRel(l, SelfRunID)) equals StencilPinsFile(l, SelfRunID).
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/loomcli/resume.go:164
    summary: The resume path that wakes a parked driver adds the stencil check, but no test fails if it is removed.
    fix: Add a row to TestResumeVerb_NamesStencilsChangedSinceTheRunStarted that resumes a parked live driver (putParkMarker plus a live driver directory row) with a pinned and edited stencil, asserting stencils_changed and one notice.
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/loomcli/stencilpins.go:57
    summary: The documented failure paths have no test; a failed parent post must keep the verb's success and still list the changed stencils.
    fix: Add a row to TestResumeVerb_NamesStencilsChangedSinceTheRunStarted whose notifyParent returns an error, asserting the envelope is ok and stencils_changed still names the edited stencil.
  - id: F7
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/stencilstore/pins.go:36
    summary: ChangedPins' name ordering and the error for a board copy that exists but cannot be read have no test row.
    fix: In TestChangedPins add a row that edits both loom-one and loom-two and wants them in name order, and a row that makes a pinned board copy unreadable through permkit.Chmod and asserts ChangedPins returns an error naming that stencil.
  - id: F8
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/loomcli/start.go:628
    summary: Every successful start posts the same parent notice again, including the live-driver no-op start that resumes nothing, so the parent orch gets a duplicate per start while resume's no-op paths warn of nothing.
    fix: Warn on start only when it spawned or woke a driver, matching resume; have runDriverSpawnAndWait report whether it put a driver to work (mustSpawn or parkedLive) and call warnChangedStencils only then, updating the start long help and loomcli/doc.go's every-successful-start sentence to match.
  - id: F9
    severity: NIT
    class: design
    origin: lens:code-correctness
    location: internal/loomcli/resume.go:135
    summary: On both resume success paths the creel post in warnChangedStencils runs while the bootstrap lock is still held, because the argument is evaluated before succeed releases the lock.
    fix: In resumeFromRunState's two driver-spawning or waking success paths, release the bootstrap lock before calling warnChangedStencils, for example by computing the success fields, releasing, then calling warnChangedStencils and output.Ok, as start already does.
---

### [MEDIUM] Pins left uncommitted after a failed first-seed commit

**Location:** internal/loomcli/sharedbootstrap.go:185
**Issue:** `commitPaths` takes the pins only when `wroteStencilPins` is true, which needs `seedErr == nil`.
A re-entry gets `ErrSeedExists`, and neither `bootstrapCommitPaths` nor `statusCommitPathspec` (`wiring.go:113`) carries the pins file, while they do re-commit the seed, status and usage directory to self-heal the same window.
Step 3's own comment says an uncommitted path fails the phase machine's precondition scan.

### [MEDIUM] Notice overclaims what lym stencil diff shows

**Location:** internal/loomcli/stencilpins.go:66
**Issue:** `runDiffOne` (`internal/stencilcli/diff.go`) renders the base recovered by stamp against the shipped default.
After a refresh restamps an untouched copy to the new default, that diff is empty although the pin changed;
a hand edit appears only through the unstamped fallback.
The Goal licenses naming `lym stencil diff`; the defect is the promise that it shows each change since the run started.

### [MEDIUM] Start's stencil check is untested

**Location:** internal/loomcli/start.go:628
**Issue:** No test reaches `withChangedStencils` on start's success envelope or the notice it posts.

### [MEDIUM] Seed commit of the pins is untested

**Location:** internal/loomcli/stencilpins_integration_test.go:29
**Issue:** `StencilPinsRel` is referenced by no test, and the bootstrap test runs with `FABRIC_SKIP_GIT`, so the commit is a no-op.

### [LOW] Parked-driver wake path's check is untested

**Location:** internal/loomcli/resume.go:164
**Issue:** The resume test drives only the Go-driver spawn path.

### [LOW] Failed parent post path is untested

**Location:** internal/loomcli/stencilpins.go:57
**Issue:** The test's recorder always returns nil, so the never-fails-the-verb contract on a failed post is not pinned.

### [LOW] ChangedPins ordering and unreadable copy untested

**Location:** internal/stencilstore/pins.go:36
**Issue:** Every row expects at most one changed name, and no row makes a board copy unreadable.

### [LOW] Start re-sends the notice on a no-op

**Location:** internal/loomcli/start.go:628
**Issue:** `runDriverSpawnAndWait` returns true for a live, working driver too, so step 7 warns and posts with nothing resumed.
`orchcli.NotifyParent` carries no topic, so unread duplicates pile up.

### [NIT] Creel post under the bootstrap lock

**Location:** internal/loomcli/resume.go:135
**Issue:** `succeed(withChangedStencils(..., c.warnChangedStencils()))` evaluates the post before `succeed` releases the lock; start releases first.

## Rejected

- code-scope's `stencils_changed` envelope key as unrequested scope: the envelope is the warning the caller sees when the orch, or batten acting for it, runs start or resume itself, so it traces to the Goal's "warns the orch".
- code-affected-set's commit-message NIT on `e02ee4e9`: the message is history a fixer cannot change without rewriting commits, and the test's own doc comment states accurately that it skips the pair's git.
