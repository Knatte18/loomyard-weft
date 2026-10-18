# [WS 003] Orch rules move out of the repository into each orch weft

`orch-rules-files` · Task · enhancement, orch, workspaces · running · PR-Gate

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

## Operator decisions (2026-10-18, later)

- Two layers of orch rules, both inlined where `ORCHESTRATOR.md` is today, hub-wide first:
  - Hub-wide rules for every orch of the hub live on Bolt, `_board/_lyx/orch/rules.md`: the repository-specific rules a general orch needs, which the shipped role template cannot hold.
  - Each orch thread's own rules live in its own worktree's weft, `_lyx/orch/rules.md`.
- For the root umbrella the own-rules file is in the prime's `_lyx`, a link to the records repository's `main-weft` worktree, so it shows inside the prime's own folder.
- `ORCHESTRATOR.md` goes, with its mentions in CLAUDE.md and the orch role stencil; its current rules move to the new files.

## Orch recommendation, not a decision

- Not an edit of the role stencil's board copy: `lyx stencil` reads such an edit as drift to port back to the shipped source, which these rules are not.
