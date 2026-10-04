# A parked label hides dormant notes from the default view

`board-parked` · Notes · design

Operator 2026-10-04, undecided ("vi tenker på det"): the board holds every idea, and a long flat list is hard to read.
Idea: a `parked` label for notes nobody means to act on for a while, hidden from the default listing and README but kept; the orchestrator proposes merges, parking and deletions at each intake, so the list does not grow unchecked.
It is a label, not a layer, so it needs only a config entry and a default filter once `board-model` lands.

Triage proposed by the orchestrator on 2026-10-04, not yet approved:

- Park candidates: `codex-engine`, `reed-slack-relay`, `semantic-index`, `lyx-mcp`, `creel`, `plugin-packaging`, `session-sync`, `hardener`, `cache-ttl-per-role`, `fabric-windows`, `fabric-monorepo-verbs`.
- Read the remaining notes as clusters, each a candidate for one bundled task when its turn comes: orch loop, ide, Webster robustness, orch in Discussion (`post-discussion-additions`, `circling-to-parent`, `parent-review-hardening`, `discussion-advisor`), Burler, glyph, fabric.
  Once `board-model` lands, these become area labels, so the viewer's area filter shows each cluster.
