# Card 2 — Loom config key and parent-review paths

**Edit:**
- `internal/loomengine#Config`
- `internal/loomengine#LoadConfig`
- `internal/loomengine/template.yaml`
- `internal/loomengine/config.go`
- `internal/loomengine/config_test.go`
- `internal/loomengine/review_test.go`

**Create:**
- `plan:internal/loomengine#LoomParentReviewDirRel` -> `func LoomParentReviewDirRel() string`
- `plan:internal/loomengine#LoomParentReviewDir` -> `func LoomParentReviewDir(l *lyxcwd.Location) string`
- `plan:internal/loomengine#LoomParentReviewLockDir` -> `func LoomParentReviewLockDir(l *lyxcwd.Location) string`

**Uses:**
- `internal/loomengine#LoomReviewsDirRel`

**Intent:**
Give loom the wait bound of decision D10 and the on-disk location of decision D3's request directories.

`Config` gains `ParentReviewWaitMin int` from `loom.yaml`'s `parent_review_wait_min`, defaulting to 60 in `template.yaml` with a trailing comment naming it the minutes Discussion-Write's gate waits for the parent's verdict before passing with a Warn.
`LoadConfig` rejects a negative value alongside the other timeout knobs; 0 is legal and expires an open request at its first evaluation.

`config.go` declares the `parent-review` segment (`parentReviewDirName`), sole declarer as for the sibling segments.
`LoomParentReviewDirRel` joins it onto `LoomReviewsDirRel`, so the round directories sit beside the Discussion-Review run directory under the run root and never under `_lyx/discussion/`.
`LoomParentReviewDir` anchors the same path on `l.AnchorPath()`.
`LoomParentReviewLockDir` is the ephemeral mirror under `.lyx` (`lyxdirs.DotLyxDirName`, the reviews segment, then the parent-review segment), where the store keeps its lock files, per the Durable-vs-Ephemeral State Invariant.

Tests: `config_test.go` covers the default, an explicit value and the negative refusal; `review_test.go` pins the three accessors' paths.

**ImpactSummary:** One new loom.yaml key with a template default and three new path accessors in loomengine; existing keys and accessors are unchanged.
