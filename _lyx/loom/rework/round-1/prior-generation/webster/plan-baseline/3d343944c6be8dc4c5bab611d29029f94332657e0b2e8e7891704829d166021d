# Card 16 — creel-docs

**Prosa:**
- `internal/creelengine/doc.go`
- `docs/overview.md`

**Intent:** Move the whole creel design into its package doc and the module map (D20, `PATTERN-documentation-lifecycle`).
`internal/creelengine/doc.go` carries the as-built stage-1 design: layering creel → shuttle → reed and the `Delivery` seam; the per-hub daemon, its lock, socket, protocol and replaced-binary exit; sender identity and its bounds; receivers; the journal, the fold, ids and retention; post and reminders with their backoff; read and its ack; inject, its TTL and privilege; interrupt; forward; replies and waiting senders; holds; pause; removed holders; orch strands in stage 1; log; the delivery loop.
It also carries the parts stages 2 and 3 build, marked unbuilt: the note request as mail answered by `written <path>` or `defer`, the orch watcher keeping only its cycle logic and holding through creel, `PATTERN-orch-pane-single-writer` becoming "only creel decides what is delivered into an agent's pane", `orchengine.QueueNotice` folding into creel, and drivers' stop reports by post.
`docs/overview.md` gains `internal/creelcli/` and `internal/creelengine/` lines in its package tree and a **creel** bullet in its Modules list naming the verbs, the per-hub daemon and that no module sends through it until `creel-notices`.
In the same commit the `creel-core` board entry's body is cut to a pointer at `internal/creelengine`'s package doc, through `lyx board upsert`.
