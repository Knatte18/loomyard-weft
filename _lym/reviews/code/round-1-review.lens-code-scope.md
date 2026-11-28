```yaml
lens: code-scope
outcome: reported
```

Lens code-scope: tracing the diff's changes to the board entry's goal points.

**Per change, the decision it traces to**

- C1, `internal/gitexec/transport.go`: adds `missingRepositoryPattern` and `namesMissingLocalPath`, makes `IsTransportFailure` fall through to them, and extends its doc comment. This builds G1, G2 and G3, choosing "treat it as unreachable in both places". Evidence: diff 003, the `transport.go` hunks at lines 68–98 and 121–124.
- C2, `internal/gitexec/doc.go`: adds two lines saying a missing repository path counts as a transport failure. Forced by C1, since the package doc describes what `IsTransportFailure` reports. Evidence: diff 003, `doc.go` at line 22.
- C3, `internal/gitexec/transport_test.go`: adds the missing-path rows and the file-header line. Forced by C1, as its test.
- C4, `internal/gitexec/transport_integration_test.go`: adds the removed-origin and empty-directory cases with real git, and updates the header and `testtiming:keep` text. Forced by C1, as its test.
- C5, `internal/fabricengine/pushclasses_integration_test.go`: the new `ARemovedLocalOrigin` row builds the test of G1. The `other`-class fixture moving from a missing path to `t.TempDir()` is forced by C1.
- C6, `internal/fabricengine/originreconcile_integration_test.go`: adds the `wantUnreachable` column and the "an origin missing on disk" row, which test G2. The no-repository rows moving to an existing directory are forced by C1.
- C7, fixture swaps from a missing path to an existing non-repository directory: `internal/cleanupcli/cleanup_integration_test.go:271`, `internal/fabricengine/add_remoteleftover_integration_test.go:96,116` and `internal/fabricengine/pushanchored_integration_test.go:281,291`. All forced by C1: each test means "an origin that is no repository", which a missing path no longer gives.

**Per decision, the changes that build it**

- G1, the push failure classification classes a removed bare origin as transport: built by C1 and tested by C5.
- G2, the origin reconcile reports unreachable instead of an error for a missing directory: built by C1 through the `lsRemotePair` and `judgeAgainstOrigin` calls of `IsTransportFailure` (`internal/fabricengine/originreconcile.go:236,625`), and tested by C6.
- G3, decide how a missing local remote is classed, either unreachable in both places or a documented network-only rule: decided as "unreachable in both places" by C1 and documented by C1 and C2.
- Tests no longer have to use a refused-connection URL: this is context, not a separate ask. C5 and C6 now use a removed local origin.
- There are no `Added after Discussion` entries.

**Findings**

None. Every change traces to a goal point or is forced by C1, and every goal point is built. No commit carries a `Pattern-Departure:` trailer, and none is needed.

Outside my lens: four tests the diff leaves untouched still point origin at a missing path, and C1 now reclassifies that path as a transport failure:
- `internal/fabricengine/add_rollback_adopt_test.go:270-271`;
- `internal/fabricengine/clone_singlerepo_integration_test.go:878`;
- `internal/battencli/lifecycle_integration_test.go:869`;
- `internal/hubreconcile/ensure_integration_test.go:776`.

That belongs to the affected-set and tests lenses.

Tool calls: none.
