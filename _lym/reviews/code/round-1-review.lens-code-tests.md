```yaml
lens: code-tests
outcome: reported
```

Restating my task: the code-tests lens, which checks which test would fail if each behaviour the diff adds or changes were broken.

**Behaviour → test that pins it**

1. `stencilstore.Pins` hashes each registry stencil's board copy with `BodyHash` and leaves out a stencil with no board copy → `stencilstore.TestChangedPins`, through the `wantPins` assertion every row makes (`loom-three` has no copy, so it is left out). Pinned.
2. `Pins` / `ChangedPins` return an error naming the stencil when a board copy exists but cannot be read → none.
3. `ChangedPins` reports an edited body, reports a removed copy, ignores a banner-only restamp and ignores a stencil installed after the pins → `TestChangedPins` rows `EditedBodyIsReported`, `RemovedCopyIsReported`, `RestampedBannerIsNotReported`, `StencilInstalledAfterThePinsIsNotReported`, `UntouchedCopiesReportNothing`. Pinned.
4. `ChangedPins` returns names in name order → none. Every row expects at most one changed name.
5. `shedrun.WriteStencilPins` / `ReadStencilPins` round trip, with an absent record read as not found → `shedrun.TestStencilPins_RoundTrip`. Pinned.
6. `shedrun.StencilPinsRel` is the right anchor-relative pathspec, and the seed commit carries the pins file → none. No test references `StencilPinsRel`. `TestSeedAndCommitBootstrap_PinsStencilsOnTheFirstSeedOnly` sets `FABRIC_SKIP_GIT=1`, so `CommitAnchoredPaths` commits nothing and a wrong pathspec, or a dropped `commitPaths` append, fails no test.
7. The first seed writes the pins and a re-entry keeps them → `TestSeedAndCommitBootstrap_PinsStencilsOnTheFirstSeedOnly`. If the `seedErr == nil` guard were removed, the second call would rewrite the pins with the edited stencil's hash and the `DeepEqual` against `wantPins` would fail. Pinned.
8. A failed `writeStencilPins` logs at Warn, returns false and adds nothing to `commitPaths` → none.
9. A resume that spawns a Go driver lists `stencils_changed` and posts one parent notice naming the stencil and `lym stencil diff <name>`. With no pins or no change, it reports nothing and posts nothing → `TestResumeVerb_NamesStencilsChangedSinceTheRunStarted`, all three rows. Pinned.
10. A resume that wakes a parked driver lists `stencils_changed` and notifies → none. The `withChangedStencils` call at `resume.go:164` can be removed with no test failing.
11. A successful `lym loom start` lists `stencils_changed` and notifies (`start.go:628`) → none. No start test writes pins, edits a stencil and asserts the envelope or a notice.
12. The no-op resume successes (done run, already running) and refusals neither warn nor list → none. This negative case is unpinned, though it holds today only because those paths do not call `warnChangedStencils`.
13. A failed pins read, failed compare or failed `notifyParent` post is logged and never fails the verb. On a failed post, the envelope still lists the changed stencils → none.
14. The production `notifyParent` wiring to `orchcli.NotifyParent(c.location, loomProgram, …)` → none. It is a seam; the substitution covers its callers.

**Findings**

1. MEDIUM · design · `internal/loomcli/start.go:628`: the start verb's stencil check has no test. The diff adds `withChangedStencils(..., c.warnChangedStencils())` to start's success envelope, and the summary names start as the path batten resumes child runs through, yet no test pins `stencils_changed` or the parent notice on start. Evidence: grep finds `stencils_changed` and `withChangedStencils` only in `resume_test.go:52` and `stencilpins_test.go:32,75`. Fix: add a start test row with pins written and a stencil edited, asserting the envelope field and one notice.
2. MEDIUM · design · `internal/loomcli/resume.go:164`: the woken-parked-driver path's stencil check has no test. `TestResumeVerb_NamesStencilsChangedSinceTheRunStarted` drives only the Go-driver spawn path (`spawnGoDriver` stub, `StateBlocked`, go seed), so the parked-driver `succeed(withChangedStencils(...))` is unpinned. Evidence: `stencilpins_test.go:51-63`. Fix: add a row to that test that resumes a parked live driver (`putParkMarker` plus a live directory row).
3. MEDIUM · design · `internal/loomcli/sharedbootstrap.go:185-187`: nothing tests that the seed commit carries the pins record. `StencilPinsRel` is referenced by no test, and the only bootstrap test sets `FABRIC_SKIP_GIT`, so a wrong pathspec (for example under the legacy `self` segment) or a dropped append passes every test. Evidence: `stencilpins_integration_test.go:29`; grep for `StencilPinsRel` in `_test.go` files finds nothing. Fix: assert `StencilPinsRel` is in the committed paths, or check `StencilPinsRel` joined to the anchor equals `StencilPinsFile` in the round-trip test.
4. LOW · design · `internal/loomcli/stencilpins.go:41-59`: the failure paths of `warnChangedStencils` have no test. A failed `notifyParent` must keep the verb's success and still list the changed stencils, and a failed read or compare must report none without failing the verb. Evidence: `stencilpins_test.go:53-56` (the recorder always returns nil). Fix: add a row whose `notifyParent` returns an error, asserting ok and `stencils_changed`.
5. LOW · design · `internal/loomcli/stencilpins.go:22-33`: a failed `writeStencilPins` (Warn, false, pins path not committed) has no test. Evidence: no test makes `Pins` or `WriteStencilPins` fail.
6. LOW · design · `internal/stencilstore/pins.go:61-66`: the `readBoardCopy` error for an existing but unreadable copy, and `ChangedPins`' name ordering, have no test row. Evidence: `pins_test.go:26-46` has no unreadable-copy row (`permkit.Chmod`) and no row with two changed names. Fix: add a row editing two stencils, which pins the order, and an unreadable-copy row.

Tool calls:
1. grep of `start_driver_test.go` for `StencilsDir` / `stencilkit`, and of all tests for `StencilPinsRel`, `withChangedStencils` and `stencils_changed`. My context lacked whether the shared receiver seeds stencils and which tests reference the new symbols.
