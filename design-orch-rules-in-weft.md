# [WS 003] Orch rules move out of the repository into each orch weft

`orch-rules-in-weft` · Note · enhancement, orch, workspaces

## Seen (2026-10-18)

- `ORCHESTRATOR.md` is a tracked file at the repository root holding the orch-only rules of this hub; the watcher inlines it into the role file's `## This repository` at every render (`RepoRulesPath` in `internal/orchcli/paths.go`).
- Its rules are the operator's choices for how lyx behaves in this hub, not knowledge about the code, so anyone cloning the repository gets them unasked.
- It is written for one orch, which no longer fits once umbrellas each have their own orch thread (`orch-any-worktree`).
- The orch role stencil already has a board copy on Bolt (`_board/_lyx/stencils/orch/orch-template-role.md`), and hub config lives in `_board/_lyx/config/`.

## Operator view, not a decision (2026-10-18)

The file breaks the concept: instructions for how something in lyx behaves do not belong in the repository, but under `_lyx`, tracked in the weft repository.
Bolt is hard to see and search from a working worktree, which the umbrella workspaces (`hub-workspaces`) solve by listing Bolt as a folder.

## Operator view, not a decision (2026-10-18, later)

The rules belong in each orch's own weft `_lyx`, not on Bolt, since an umbrella's orch does not want the same rules as another's.

## Orch recommendation, not a decision

- Each orch thread's rules live in its own worktree's weft, `_lyx/orch/rules.md`, inlined where `ORCHESTRATOR.md` is today; `ORCHESTRATOR.md` and its CLAUDE.md mentions go.
- For the root umbrella that is the prime's `_lyx`, a link to the records repository's `main-weft` worktree, so the file shows inside the prime's own folder with no Bolt folder needed.
- No hub-wide rules file on Bolt.
- Not an edit of the role stencil's board copy: `lyx stencil` reads such an edit as drift to port back to the shipped source, which these rules are not.
