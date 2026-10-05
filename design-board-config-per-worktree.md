# board.yaml is read per worktree though the board is one per hub

`board-config-per-worktree` · Task · bug, board, config · running · Plan-Burler

The board is one per hub (`<hub>/_board`), but its config, `board.yaml` (types, labels, README settings), is read from each worktree's own `_lyx/config/`.
Seen 2026-10-05: after the hub added the `feature` type in the prime's config, the landing of `lyx-skills-and-stencils` wrote to the board from its pair, whose copy predates the change, and re-rendered README with every `feature` note under "Other"; the hub re-rendered from the prime.
Every pair created before a vocabulary change does the same until it is torn down.

Precedent: `fabric.yaml` already lives once per repo at `<BoardDir>/_lyx/config/fabric.yaml`, because the junction set is a repo-wide fact every worktree reconciles against (`internal/fabricengine/doc.go`).
`board.yaml` describes the one board, so the same holds.

Proposal: `board.yaml` moves beside `fabric.yaml` in `_board/_lyx/config/`, and every `lyx board` verb reads it from there, whatever worktree it runs in.
Check which other config files are repo-wide facts the same way before generalizing; `config-overlay` is the general version, `config-reaches-parent` the related bug.
