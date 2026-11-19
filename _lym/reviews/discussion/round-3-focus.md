---
round: 3
focus:
  - "D1 and D3 in decision-record.md: the board's local branch name against fabric's readers. `CloneHub` builds the board with `ensureBoardWorktree(weftPath, warpBranch, boardDir)` (internal/fabricengine/clone.go:328, boardweft.go:24-51), and `primaryWeftBranch` (internal/fabricengine/cleanup.go:337-351) derives the prime's records branch from the board's checked-out branch for the orphan sweep (cleanup.go:134) and `resolveManagedBranch` (destroy.go:828, 847). Settle whether the record names, for a code default branch other than `main`, which branch the board is created on in the `--single-repo` and one-URL clone paths, what `primaryWeftBranch` reads in a one-repository hub, and that a two-repository hub's answer is unchanged."
  - "Acceptance criteria in decision-record.md: settle whether one criterion runs the clone, a second-machine one-URL clone and the orphan sweep over an origin whose code default branch is not `main`, so a fixture with a `main` default does not hide a mismatch between the board branch and the prime's records branch."
  - "Any text round 2's fix adds to D1, D3 or the acceptance criteria: check each added claim about git or fabric behaviour for a stated result and pin, each new verb or routing edge for what it can skip and what bounds it, and that no added text leaves a fact for Plan or an implementer to find out."
---

The focus targets the one open design area.
Nothing else is promoted: the remaining round-1 findings read as resolved.
