# Card 9 — `lyx loom status` shows the reviewer wait

**Create:**
- `internal/loomcli/reviewstatus.go`
- `internal/loomcli/reviewstatus_test.go`

**Edit:**
- `internal/loomcli#loomCLI.specFor`

**Prosa:**
- `contracts/specs/loom-status-spec.md`

**Uses:**
- `internal/shedverbs#Hooks`
- `plan:internal/parentreview#Store`
- `plan:internal/parentreview#Store.WaitNote`
- `internal/loomengine#LoomParentReviewDir`
- `internal/loomengine#LoomParentReviewLockDir`

**Intent:**
Fill card 8's `Waiting` hook for loom, so `lyx loom status` reports a run waiting on its reviewer (decision D9).

`reviewstatus.go` holds the loom hook: it builds a `parentreview.Store` over this worktree's `LoomParentReviewDir` and `LoomParentReviewLockDir` and returns `Store.WaitNote()`, prefixed `parent review: `.
A missing parent-review directory is an empty note, never an error.
`specFor` sets `Hooks.Waiting` to it, so the lightweight `status` wiring carries it with no module config.
Only the latest round is reported, so a superseded round never shows.

`reviewstatus_test.go` writes round fixtures through the store and asserts the note for an open undelivered request, an open delivered one, and none for a verdicted, expired or superseded round or an absent directory.
`loom-status-spec.md` documents the `waiting` key and when it appears.

**ImpactSummary:** `specFor` gains one hook assignment read only by `status`; every other verb's spec is unchanged.
