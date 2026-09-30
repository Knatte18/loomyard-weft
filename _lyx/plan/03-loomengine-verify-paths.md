# Card 3 — loomengine: scratch accessors for the verify marker and output

**Create:**
- `internal/loomengine#LoomVerifyPendingPath`
- `internal/loomengine#LoomVerifyOutputPath`

**Edit:**
- `internal/loomengine#TestLandingAndApprovalAccessors`
- `cmd/lyx#transientSet`
- `cmd/lyx#TestConstructorAnchoring_Unanchored`
- `cmd/lyx#TestConstructorAnchoring_SubpathAnchored`

**Uses:**
- `internal/loomengine#LoomScratchDir`

**Intent:**
Add two ephemeral path accessors to `internal/loomengine/config.go`, beside `LoomApprovalPath` and built the same way on `LoomScratchDir` (never re-joining the `.lyx` literal, per the Lyxdirs Single-Declarer Invariant):
`LoomVerifyPendingPath` returns the pending-verify marker's path, and `LoomVerifyOutputPath` returns the path the post-merge verify command's combined output is written to.
Declare each filename as a private constant next to `loomApprovalFileName` — `verify-pending` for the marker and `verify-output.log` for the output.
Both files are never tracked, so they live under loom's scratch directory per the Durable-vs-Ephemeral State Invariant, and the doc comments say so, name the landing producers as their consumer, and state that no other package may construct either path.

Pin both accessors in the existing guards the way `LoomApprovalPath` is pinned:
extend `TestLandingAndApprovalAccessors` with an exact-path assertion for each (`<anchor>/.lyx/loom/verify-pending` and `<anchor>/.lyx/loom/verify-output.log`);
add both to `cmd/lyx`'s `transientSet` so `TestNoTransientsUnderLyx` proves they resolve under `.lyx`;
and add an `assertPath` line for each to both `TestConstructorAnchoring_Unanchored` and `TestConstructorAnchoring_SubpathAnchored`.

**ImpactSummary:** Additive: two new accessors and two new filename constants; the edited tests only gain assertions.

**Commit:** 3: loomengine: add scratch accessors for the verify marker and output
