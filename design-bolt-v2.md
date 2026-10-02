## Goal

Bolt becomes the roadmap: one store with a numeric tier and a `type`, a README rendered like roadmap.md, `lyx board find` and a compact listing.
The operator finds bolt's layout poor; this task is the layout and store only, and retiring `manifest/` is the separate task `manifest-retire`, which depends on this one.

## Must not break running work

Other runs land while this one does, and their landings write to bolt with the deployed lyx.
The store migrates itself on first use by the new binary, and the existing task and notes verbs keep working (or map onto the new store) so no other run's landing breaks.
## From `bolt-roadmap-tiers`: Bolt as the roadmap: three numbered tiers in one store, rendered like roadmap.md

### Why

Operator 2026-10-01: `manifest/roadmap.md` is very readable and bolt's README is not.
Bolt should follow the roadmap's principle; everything in it matters to the orchestrator, since the future direction shapes how Planned work is reviewed.

### Tiers

1. Concretized and claimable (rendered Planned).
2. Planned next but not concretized; not claimable until promoted (Next Up).
3. Loose ideas the operator has thought about, as in roadmap.md today (Someday).

### Storage

- One store, not `tasks.json` + `notes.json` (and not three files); every entry carries a numeric `tier`.
- The data holds only the number; the renderer alone owns the section names.
- Claimable means tier 1; promotion lowers the number (3 → 2 → 1, skipping allowed); new ideas default to tier 3.
- The task and notes verb groups collapse onto it; migrate the two existing files.
- Operator also wants a `type` field (e.g. design, bug, idea, chore) alongside the tier; settle the type set in Discussion.

### Rendering

- README like roadmap.md: one section per tier, in tier order, each with a one-line meaning.
- One `1.` item per entry with **title**, slug, one or two what/why sentences and the design link; no per-entry headings, no numeric ids.
- Optionally live state, producer and PR on running tier-1 entries.
- Design docs render as markdown: today a body written with single newlines shows as one paragraph.

### Agent ergonomics (folded in from `board-agent-ergonomics`)

The hub fell back to jq on `notes.json` because `lyx board list` shows tasks only and nothing searches.

- `lyx board find <text>` over every entry.
- A compact human listing (tier, slug, title) besides the JSON.
- The `get`/`remove` payload shapes in `--help`.
- A skill or doc telling agents how to use bolt (see also `lyx-mcp`).

### Also folds in

- `board-prune-done`: one verb removing every done entry, its design doc and dangling `depends_on`.
- After the manifest triage, roadmap.md's Someday moves into bolt and bolt becomes the only roadmap (`manifest-to-bolt`).
- Content job, not code: rewrite entry briefs as what/why during the review.


