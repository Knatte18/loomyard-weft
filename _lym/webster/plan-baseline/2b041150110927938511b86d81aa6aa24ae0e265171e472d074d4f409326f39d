# Card 13 — shared-bolt-pull

**Create:**
- `plan:internal/boardengine#PullBolt` -> `func PullBolt(boardDir string) error`

**Edit:**
- `internal/battencli#hubOwnerSeams`
- `internal/battencli#hubCancelDeps`

**Delete:**
- `internal/battencli#pullBolt`

**Uses:**
- `internal/boardengine/doc.go`

**Intent:** Move batten's pull-first write of nothing into `boardengine` as `PullBolt`, body unchanged, so `tendercli`, which `battencli` imports and so cannot import back, reads the owner from a Bolt it fetched first (card 14).
It keeps the `BOARD_SKIP_GIT` and `BOARD_SKIP_PUSH` skip and the `ErrOwnerOriginUnreachable` wrap of a failed fetch.
`hubOwnerSeams` and `hubCancelDeps` call it in place of `pullBolt`.
A pure move: the owner and cancel tests that drive these seams cover it.

**ImpactSummary:** One function moves packages; its two callers in battencli switch to the new name, behaviour unchanged.
