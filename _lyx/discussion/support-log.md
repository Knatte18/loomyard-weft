# Support log — gate-fewer-new-binaries

## Interview

A darn run holds no interview: the writer worked from the board entry and a reading of the code, with no operator present.

## Rejected alternatives

- Keep the unit tier as a fast fail before the integration tier.
  Rejected: the tagged run reports the same unit failures, and the unit tier costs its own wall time and its own test binaries on every passing verify, which is the common case.
- Pin `LYX_TRACE_ID` to a constant instead of removing it.
  Rejected: a constant reads as a real trace id in any log a test writes, and removing it is the same stable value to Go's cache.
- Give the prebuilt `lyx` a fixed path per hub or per slot.
  Rejected: a path that does not change with the binary lets Go reuse a cached pass for a test that runs a changed `lyx` whose packages its own test binary does not import.
- Record failed tests inside `lyx gate test` and re-run them first on its next call.
  Rejected: the verify gate already reads the failing tests from the Publish failure record and only appends them in the wrong place; moving them to the front is the whole fix.
- Reuse cached results across worktrees for tests that open files of the tree.
  Rejected: Go keys such a result on each file's absolute path and modification time, which a fresh worktree changes; no gate-side change reaches that.

## Review rounds

_No rounds yet._

## Question ledger

- Is any test excluded by the `integration` tag? Resolved (self-pick): no; the `!integration` files are TestMain shims whose tagged counterparts do the same setup and more.
- Why do packages miss the cache on an unchanged tree? Resolved: `GODEBUG=gocachetest=1` showed the input ID changing on every run of `./internal/gateslot`; its test log reads `LYX_TRACE_ID`, and with that variable pinned the second run was cached. The prebuilt `lyx` path is the second per-run value, read by the lyxbin packages.
- Who changes the hub's verify command? Resolved (self-pick): the parent, since `darn.yaml` is hub-wide config on Bolt.
- How long does the prebuilt store keep a binary nobody uses? Resolved (self-pick): two hours since its last use, far beyond any gate run.
- May the writer run module-wide gate runs for the measurement? Resolved (self-pick): yes, the board entry asks for the before and after numbers, which only module-wide runs give.
