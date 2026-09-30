# Card 14 — fabric-sibling-dirty-sentinel

**Create:**
- `plan:internal/fabricengine#ErrPairSiblingDirty` -> `var ErrPairSiblingDirty error`
- `internal/fabricengine/remove_siblingdirty_integration_test.go`

**Edit:**
- `internal/fabricengine#refuseDirtyWeftWorktree`

**Intent:**
Let a caller outside the fabric vocabulary owner set recognise `Remove`'s refusal of a pair whose other side carries uncommitted changes, so batten can name its own remedy instead of fabric's `--force`.

Declare `ErrPairSiblingDirty` in `remove.go` with `errors.New`, its message and doc comment worded without naming either side, since callers such as `battencli` are outside the Fabric Vocabulary Invariant's owner set.
`refuseDirtyWeftWorktree` wraps it with `%w` in its dirty refusal, keeping today's text and remedy; `nameStrandedPortalTeardown` already wraps with `%w`, so `errors.Is` holds on `Remove`'s returned error.
The unreadable-status refusal is not wrapped: it is not a dirtiness answer.

Test in `remove_siblingdirty_integration_test.go`, over a hubforge hub: a pair whose `_lyx` tree holds a new, untracked file inside a not-yet-tracked `_lyx/shed/<slug>/drive-reports/` directory (the uncommitted stop report the done-wait Decision rests on) refuses `Remove` (no force) with an error satisfying `errors.Is(err, fabricengine.ErrPairSiblingDirty)`, and a pair dirty only on the task side does not satisfy it.

**ImpactSummary:** One new sentinel wrapped into an existing refusal; the refusal's text and every caller's behavior are unchanged.
