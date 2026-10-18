# Hub orch rules move out of the repository into Bolt

`orch-rules-in-weft` · Note · enhancement, orch, work-laptop

## Seen (2026-10-18)

- `ORCHESTRATOR.md` is a tracked file at the repository root holding the orch-only rules of this hub; the watcher inlines it into the role file's `## This repository` at every render (`RepoRulesPath` in `internal/orchcli/paths.go`).
- Its rules are the operator's choices for how lyx behaves in this hub, not knowledge about the code, so anyone cloning the repository gets them unasked.
- It is written for one orch, which no longer fits once umbrellas each have their own orch thread (`orch-any-worktree`).
- The orch role stencil already has a board copy on Bolt (`_board/_lyx/stencils/orch/orch-template-role.md`), and hub config lives in `_board/_lyx/config/`.

## Operator view, not a decision (2026-10-18)

The file breaks the concept: instructions for how something in lyx behaves do not belong in the repository, but under `_lyx`, tracked in the weft repository.
Bolt is hard to see and search from a working worktree, which the umbrella workspaces (`hub-workspaces`) solve by listing Bolt as a folder.

## Orch recommendation, not a decision

- Hub-level rules for every orch thread in the hub live in `_board/_lyx/orch/rules.md` on Bolt, inlined where `ORCHESTRATOR.md` is today; `ORCHESTRATOR.md` and its CLAUDE.md mentions go.
- An umbrella may add rules of its own in its weft, `_lyx/orch/rules.md` in its pair, inlined after the hub rules for that umbrella's thread only.
- Not an edit of the role stencil's board copy: `lyx stencil` reads such an edit as drift to port back to the shipped source, which hub rules are not.
