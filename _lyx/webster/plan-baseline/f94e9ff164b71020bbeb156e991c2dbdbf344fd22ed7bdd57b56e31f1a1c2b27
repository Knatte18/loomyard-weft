# Card 1 — buildvcs-commit-time

**Edit:**
- `internal/buildvcs#Identity`
- `internal/buildvcs#identityOf`
- `internal/buildvcs/buildvcs_test.go`
- `internal/buildvcs/doc.go`
- `//cmd/lyx/testdata/test-budget.yaml`

**Create:**
- `internal/buildvcs#Identity.Clean`
- `internal/buildvcs#Identity.Label`
- `internal/buildvcs#shortRevisionLength`

**Intent:** `Identity` gains a `Time time.Time` field, JSON key `vcs_time`, and `identityOf` fills it from the `vcs.time` build setting (RFC 3339), leaving it zero when the key is absent or unparseable (D1: the commit time stands beside the revision).
`Clean` reports a stamped, unmodified build: a non-empty `Revision` with `Modified` false; it is the one predicate D2's mode choice and D1's forced-refresh stamping both read.
`Label` renders the identity as `hubreconcile`'s `revisionLabel` does today, `unknown` for an empty revision, the first `shortRevisionLength` characters otherwise, with a `-modified` suffix when `Modified` is set; `shortRevisionLength` moves here with it so card 10 can delete the `hubreconcile` copy.
The package stays stdlib-only (`PATTERN-leaf-packages`); `doc.go` names the three readers of the stamp.
`TestIdentityOf` gains rows for a parseable `vcs.time`, a missing one and a malformed one; a new `TestIdentity_CleanAndLabel` table covers the four `Clean` cases and the label forms, so `internal/buildvcs` rises by one in the test budget.

**ImpactSummary:** Adds a field to a struct every `Identity{}` literal in the tree constructs keyed, so no caller breaks; `hubreconcile`'s stamp JSON gains an extra key its reader ignores.
