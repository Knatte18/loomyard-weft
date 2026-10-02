## Goal

Bolt becomes the roadmap: one store with a numeric tier and a `type`, a README rendered like roadmap.md, `lyx board find`, a compact listing; then `manifest/` is deleted.

## manifest/ retirement

The triage was done in the bolt review of 2026-10-01/02; what survives is already in bolt.
This task moves the durable reference content into package docs, fixes inbound links and deletes the folder (Millhouse has no ownership of `manifest/`).

- `manifest/HANDOFF.md`: the hub run loop goes to the `internal/orchcli` package doc; the rest is spent.
- `hardener.md`: the text-review vs behaviour-review distinction and the respawn rationale go to `internal/treadleengine`'s doc.
- `fabric-windows-verification.md`: the Linux-only caveat goes to the `internal/fabricengine` and `internal/fslink` docs.
- `webster-parallel-execution.md`: the 42-card case study and the index-race rationale go to `internal/websterengine`'s doc.
- `review-finding-classification.md`: the convergence case study goes to `internal/burlerengine`'s doc.
- `curation-triage.md`: one line in the boardengine doc ("anyone adds notes, the orchestrator curates"); the automation is struck.
- Inbound links: `internal/treadleengine/doc.go`, `internal/fabricengine/livestate_doc_test.go`, `contracts/specs/loom-plan-spec.md`, `docs/overview.md` (lifecycle text and design links), `crucible/README.md`, `crucible/review-prompt-template.md`, CLAUDE.md's Task completion rule, `.claude/agents/mill-done-gate-fixer.md`.

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


## From `manifest-to-bolt`: Move everything in manifest/ into bolt and retire the folder

Operator 2026-10-01: LYXHUB takes over all development of loomyard; everything in manifest/ goes into bolt.
The operator also says much of it is outdated: triage each roadmap Someday item with the operator (keep as tier 3 or tier 2, merge, or strike) and carry its designs/<name>.md over as the bolt entry's design doc.
Depends on bolt-roadmap-tiers for the tiers and the readable render.
Afterwards: delete manifest/, and update CLAUDE.md (Task completion section names manifest/roadmap.md and manifest/designs/), docs/overview.md's Documentation Lifecycle and any stencil or test that names manifest/.