# Card 1 — origin-parent-worktree

**Edit:**
- `internal/fabricengine/origin.go`
- `internal/fabricengine/add.go`
- `internal/fabricengine/doc.go`
- `internal/fabricengine/origin_test.go`
- `internal/fabricengine/origin_integration_test.go`

**Intent:**
`fabricengine.Origin` gains a write-once `ParentWorktree string` field, JSON key `parent_worktree` with `omitempty`, holding the `WorktreeName` of the worktree `Topology.Add` ran from.
`Add` writes it in the same `WriteOrigin` call and the same weft commit as `ParentBranch` (step 10c), as `l.WorktreeName` of the acting location, so a pair created from the prime records the prime's worktree name and a pair created from another pair records that pair's slug.
Batten's Worktree-Create already goes through `Add` from the prime, so it needs no change of its own.
`lyx loom start`'s legacy-worktree origin write in `internal/loomcli/sharedbootstrap.go` is left as it is: it constructs `Origin{ParentBranch: parent}`, so the new field stays empty there, because a pre-origin pair's creator is unknown.
A record written before the field existed decodes with it empty.

Update `Origin`'s doc comment and its field docs, `add.go`'s file and `Add` doc comments (which say "parent-branch provenance"), and the origin-record paragraphs in `doc.go` so they name both fields and state that the parent worktree is what a run's parent agent name is resolved from.

Tests: extend `origin_test.go` with a round-trip asserting `parent_worktree` encodes and decodes, and that a record without the key decodes with `ParentWorktree` empty.
Extend `origin_integration_test.go` so a pair added from the prime records the prime's worktree name, and a pair added from inside that pair records the first pair's slug.

**ImpactSummary:** Adds one field to fabric's origin record and writes it at pair creation; existing readers ignore it and old records decode with it empty.
