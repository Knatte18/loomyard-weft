---
round: 1
focus:
  - "internal/heddleengine/render.go, target.go: does every place that assumed the root as the file-target location now use the anchor (CLAUDE.md and .claude/CLAUDE.md import check, codex-file opening from AGENTS.md, final join of each File), with the temp directory still safe for the rename onto the target?"
  - "ReRender lists in target.go and the take-in path in seam.go: do they name the same paths under the anchor, and does a worktree with no anchor keep the old root-relative paths?"
  - "internal/fabricengine/warpexclude.go and worktreeseam.go: do the exclude lines name the anchor subpath for each rendered file, in the right form for git (slashes, no leading separator), and is the anchor carried by the worktree event from where the caller already has it?"
  - "Anchor resolution: is the anchor resolved at one place that callers of the render request's Root share, and does a missing or empty anchor fall back to the root without a path-join error?"
  - "Tests: is there coverage for both a worktree with an anchor and one without, in heddleengine and in the fabric exclude path, and do the edited tests keep their prior coverage rather than duplicate it?"
  - "PATTERN.md and pattern/ edits, doc.go, config.go, template.yaml, junction.go and heddlecli/verbs.go: does each change follow from the anchor move, and does the commit's summary claim match the diff?"
---

Seeded from the decision record's Goal and Design list and the single commit's stat.
