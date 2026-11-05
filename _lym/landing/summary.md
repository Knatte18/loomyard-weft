# fabric clone --reset: check every argument before removing the hub

`lym fabric clone --reset` used to remove the existing hub first and only then probe the remotes.
A refusal from the probe, such as a `--shortname` that disagrees with the recorded one, therefore arrived after the hub was already deleted.
Now every argument is checked against the remotes before anything is removed, so a refusal leaves the existing hub untouched.

Closes #67

## What changes

- `CloneHub` in `internal/fabricengine/clone.go` removes the existing hub only after the pre-hub probe has checked the warp binding, the shortname and the old-order guard.
  This holds for the one-argument and the two-argument form.
- The two-argument form still runs its offline checks before touching the network.
  With `--reset` that offline check is now `checkResetTarget`, which refuses a target that is not a fabric hub and removes nothing.
  `resetHub` calls the same function before it removes anything.
- An explicit `--subpath` is also checked before the removal.
  A subpath that disagrees with the anchor recorded on the weft, or that names a directory absent from the warp, is refused while the old hub is intact.
  The new `internal/fabricengine/resetsubpathcheck.go` does this against staged shallow clones of both remotes, never against the hub being replaced.
  The staged clones live in a throwaway directory under the working directory and are removed before the check returns.
- `recordsprobe.go` gains `fetchWeftBranchTip`, which the probe's main-weft lookup and the subpath check share.
- `cmd/lym/destructiveguard_test.go` allowlists the staging directory's `os.RemoveAll` in `resetsubpathcheck.go`.
- `doc.go` and the `CloneHub` doc comment describe the new ordering.

## Design decisions

- No argument is exempt from the ordering: the decision was that `--subpath` is checked before the removal too, because a refusal after deletion for any argument is the same data loss.
- The subpath check reads the warp's tree from a fetch into a staging location, not from the hub being replaced.
- The check applies the same anchor rules `CloneHub` applies after its own clone:
  the prime's weft branch supplies the recorded anchor when the remote carries it, the weft's default HEAD otherwise, and the legacy board marker only when the weft's default branch is the warp's branch.
  The staged check can drift from the real clone if those rules change in one place only.
- The staging check runs only when `--reset`, an explicit `--subpath` and an existing hub all hold.
  Otherwise the clone's own post-clone anchor checks still apply.
- The one-argument form still fails offline with `probe weft <url>:` and the two-argument form with `hub already exists`; this asymmetry is unchanged.

## To check by hand

- Run `lym fabric clone --reset` with a `--shortname` that disagrees with the recorded one, against a hub in the old layout, as in the report on #67.
  Confirm the refusal leaves the hub directory intact.
  Only the integration tests have exercised this; it has not been run against real remotes.
- `recordedAnchorOnWeft` in `internal/fabricengine/resetsubpathcheck.go`: confirm its prime-branch, default-HEAD fallback and gated legacy-marker reads are each covered by a test.
  `TestCloneHub_ResetSubpathCheckReadsRecordedAnchor` in `warpbinding_clone_integration_test.go` is the covering test, but a review left the coverage open.
- `checkSubpathBeforeReset` in `resetsubpathcheck.go`: confirm that the comments and docs describing the subpath check match the code, as a review found an earlier description of it contradicting the diff.
