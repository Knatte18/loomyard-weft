# Card 2 — Loom config key and parent-review paths

**Edit:**
- `internal/loomengine#Config`
- `internal/loomengine#LoadConfig`
- `internal/loomengine/template.yaml`
- `internal/loomengine/config.go`
- `internal/loomengine/config_test.go`
- `internal/loomengine/review_test.go`

**Create:**
- `internal/loomengine#LoomParentReviewDirRel`
- `internal/loomengine#LoomParentReviewDir`
- `internal/loomengine#LoomParentReviewLockDir`

**Uses:**
- `internal/loomengine#LoomReviewsDirRel`

**Intent:**
Give loom the wait bound of decision D10 and the on-disk location of decision D3's request directories.

`Config` gains `ParentReviewWaitMin int` from `loom.yaml`'s `parent_review_wait_min`, defaulting to 60 in `template.yaml` with a trailing comment naming it the minutes Discussion-Write's gate waits for the parent's verdict before passing with a Warn.
`LoadConfig` rejects a value below 1 in a check of its own, not the sibling timeout loop, whose "use 0 to defer to shuttle's run_timeout_min" way forward does not hold for this key.
Its refusal names `attempts: 0` on Discussion-Write's `parent-review` gate entry as the way to turn the review off, which is the one off switch: a 0 bound would still open a request and notify the parent of a request that expires at its next evaluation.

`config.go` declares the `parent-review` segment (`parentReviewDirName`), sole declarer as for the sibling segments.
`LoomParentReviewDirRel` joins it onto `LoomReviewsDirRel`, so the round directories sit beside the Discussion-Review run directory under the run root and never under `_lyx/discussion/`.
`LoomParentReviewDir` anchors the same path on `l.AnchorPath()`.
`LoomParentReviewLockDir` is the ephemeral mirror under `.lyx` (`lyxdirs.DotLyxDirName`, the reviews segment, then the parent-review segment), where the store keeps its lock files, per the Durable-vs-Ephemeral State Invariant.

Tests: `config_test.go` covers the default, an explicit value, and the refusal of 0 and of a negative value; `review_test.go` pins the three accessors' paths.

**ImpactSummary:** One new loom.yaml key with a template default and three new path accessors in loomengine; existing keys and accessors are unchanged.
