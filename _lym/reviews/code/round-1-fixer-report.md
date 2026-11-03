# Fixer report — code review round 1

F10 is a `decision`-class finding and went to the parent, so it is left out of this report.
The parent recorded the decision (creel #561): the config write checks the dir, and `LoadConfig` keeps accepting a missing one.

## Changes

- **F1** (fixed as proposed, in the shared commit): `captureReading` in `internal/driftengine/seats.go` now prints the directory `gateslot.ResolveCommandDir` resolves, shown relative to the worktree root, whole when it lies outside the root, and never as empty backticks.
  `BaselineInputs` gained a required `AnchorDir`, which `internal/loomcli/driftwiring.go` fills.
  `internal/driftengine/seats_test.go` gained a `AnchorDir` required-field row and a table of exact `capture_command` values for no dir, `.`, a relative dir and an absolute dir outside the root.
- **F2** (fixed as proposed): `stepPairDirNamesTheTaskWorktree` re-anchors its pair at `backend`, asserts the anchor differs from the root, and asserts `PairDir` equals `WorktreePath()`.
  `TestReadNoticeChild` asserts `WorktreePath()` too.
- **F3** (fixed as proposed): two `TestDispatchHubWideBoard` editor-save rows for drift: a save with `capture.dir` missing in the prime is refused with the hub file unchanged and the staging copy kept, and a save over a file whose dir already failed the same check is written.
  `assertDarnSaveRefused` now wraps a module-taking `assertSaveRefused`.
- **F4** (fixed as proposed, with a refactor the review did not name):
  - `RunCompare` and `RunSideCapture` have rows with the anchor at `<root>/dotnet` and `Dir: dotnet`.
  - The request constructions moved into helpers: `captureRequest` and `compareRequest` in `internal/driftcli/wiring.go`, `driftCaptureRequest` and `driftCompareRequest` in `internal/loomcli/driftwiring.go`.
    Tier 0 tests pin the helpers' `WorktreeDir` and `AnchorDir` over a subdirectory anchor.
  - `TestDriftDeps_RunWhereTheStepsDirResolves` (tier 1) runs the real `driftDeps` seams over a `backend`-anchored hub with no dir and with `dir: backend`.
- **F5** (fixed as proposed): `TestCheckDirs` in `internal/driftengine/config_test.go` has a row seeding a compare with a missing dir.
- **F6** (fixed as proposed): `missingDirRefusal` branches on `run.Site`, through new `siteCapture`, `siteSideCapture` and `siteCompare` constants.
  The side capture's way forward is `lym drift capture --side`.
  A side-capture row asserts it, and that `lym loom resume` is absent.
- **F7** (fixed as proposed): `internal/driftcli/doc.go` names the worktree root as the join base and the anchored directory as the start of the capture child and of a step with no dir.
- **F8** (fixed as proposed): `internal/gateslot/config_test.go` `TestCheckDirs` has a row whose tier dir names a regular file.
- **F9** (fixed as proposed, the reword option): the `ResolveCommandDir` comment and `internal/gateslot/doc.go` no longer claim one rule, and say the gate run path reaches the same directory through its own default-and-join.
  `internal/driftengine/doc.go` got the same correction after the self-check, plus the side capture's way forward.

Commits: F2, F3, F5, F7, F8 and F9 (gateslot) each have their own.
F1, F4 and F6 share `seats`, the `loomcli` and `driftcli` wiring, and the capture and compare tests, so one commit carries the three.
F4's integration rows and the `driftengine` doc each have a follow-up commit.

Gate: tier 0 and tier 1 over `driftengine`, `driftcli`, `loomcli`, `gateslot`, `configcli` and `battencli`, green; tier 0 and tier 1 over `loomcli` and `driftengine` again after the self-check changes, green.

## Self-check

Lens reports: `round-1-fixer-report.lens-<lens>.md`, one per lens. No fork returned no report.

- fix-consistency, LOW: `internal/driftengine/doc.go` still said "gateslot's one rule": fixed as proposed, and its way-forward sentence got the side capture.
- fix-consistency, LOW: a stale list of told inputs may sit in the `Deps.BaselineTable` doc or `driftengine/doc.go`: disputed, premise false (`internal/driftengine/deps.go:37` and `doc.go:68` list no inputs).
- fix-code-correctness, LOW: `TestReadNoticeChild` cannot fail on a revert of the seam: disputed, every fix worse (see Disputed).
- fix-code-correctness, LOW: no test links the `driftDeps` closures to the request helpers: fixed with my own change, the tier 1 integration rows over the real seams, in place of a unit test of the closures.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

- **Self-check fix-consistency #2**, premise false.
  The reviewer did not open the docs it names.
  `internal/driftengine/deps.go:37` reads "BaselineTable builds the Drift-Baseline agent's table; scopeNote is non-empty when the scope is already recorded", and `doc.go:68` names the builders and stencils only, so neither lists `BaselineInputs`' fields.
- **Self-check fix-code-correctness #1**, every fix worse.
  The premise holds: `noticeTaskWorktree` builds a bare `.git` stub with no anchor marker, so `AnchorPath()` equals `WorktreePath()` there.
  A subdirectory-anchored task worktree needs a hub, which the Test Tier Purity Invariant keeps out of an untagged tier 0 file.
  The revert is pinned by the tier 1 step `PairDirNamesTheTaskWorktree`, which this round made fail on a revert.

## Deferred

None.
