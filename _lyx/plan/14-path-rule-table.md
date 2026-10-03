# Card 14 — path-rule-table

**Create:**
- `cmd/lyx/pathrules_test.go`

**Delete:**
- `cmd/lyx/constructoranchoring_test.go`
- `cmd/lyx/notransients_test.go`

**Uses:**
- `internal/testkit/locationkit/locationkit.go`
- `internal/lyxdirs/dirs.go`

**Intent:** Merge the two path guards into one table whose expectations come from the rule (D7).
Each row names a constructor, a closure calling it on a `*lyxcwd.Location`, and its class: durable `_lyx`, transient `.lyx`, or hub-anchored.
The table holds the union of the constructors both deleted files exercise, with no expected path literal on any row.
The test runs every row against an unanchored and a subpath-anchored `locationkit.Location` and asserts the rule: a durable path sits under `AnchorPath()/_lyx`, never ends in `.lock`, has no `pause` base and no `prompts` segment;
a transient path sits under `AnchorPath()/.lyx` and never under `_lyx`;
a hub-anchored path sits under the hub.
The mirrored-pairs check (a transient path equals its durable sibling with the `_lyx` segment rewritten to `.lyx`) is kept over pairs drawn from the same table.
Every path is still computed by pure joins, so the file stays untagged.
The deleted files' comments on the tautological `planparser`/`pattern` rows and the proofs living elsewhere carry over to the matching rows.
Prove it by moving one transient constructor's row to the durable class (and, separately, pointing one at `_lyx`), watching it fail, and restoring it;
name the proof in the commit body.
What this lets through, bounded per D7: a constructor nobody adds to the table is unguarded, as today, and the exact subpath under its root is no longer pinned.

**ImpactSummary:** `cmd/lyx` tests only; two path tables collapse into one rule-driven table over the same constructors.
