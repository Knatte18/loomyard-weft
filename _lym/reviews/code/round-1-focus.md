---
round: 1
focus:
  - "internal/hubgeom/beammodes.go readBeamMode: the board entry asks for the beam's own open pull request. Does the landing record's PullRequest stay set after that pull request is merged or closed, does the record carry anything that tells these states apart, and what link does the board then show?"
  - "internal/hubgeom/beammodes.go against how runs resolve these settings: do the chain, the anchors, the parent walk, the treatment of an absent worktree, the autonomy value mapping and the source attributed to each value match what a member's run reads? Include nested beams, a beam with no origin record, and a SetBy naming an intermediate non-beam worktree."
  - "Every production path into boardengine.RenderToDisk (board write, owner write, Bolt replay, detached push child, anything rendering a temporary or non-Bolt directory): is Config.Outputs the source of the Outputs, is the reader installed in that process, and is the directory handed to primeOfBoard one it can resolve a prime from?"
  - "Side effects and failure paths of reading during a render: what do ReadOriginFor, ReadLanding and its lock path, and the config lookups do under the board's write lock? Can they block, deadlock, spawn processes, or create or write files in another worktree, and what does a render on each failure write over the previous board files?"
  - "internal/boardengine/beammodes.go modeHeader, modeLines and everyPlaceHubWide: check the header and legend against what the lines show when a beam failed to read or is absent on this machine, when no beam is open, for done beams and non-root-child beams, and for the board entry's wording on stating the hub-wide value at the top. Does each inheritable value show whether it is inherited or the project's own, as the Added after Discussion entry asks?"
  - "internal/boardengine/render.go: the mutation report's survivor in boardView.fileHeader, and the diff's tests reading only README.md. Which behavior of the per-project files (project-<slug>.md, project-root.md) does no test check?"
  - "_lym/landing/summary.md Open risks and Merge danger: the committed board files now depend on the rendering machine's worktrees and records. Check the two-way call and the blast radius 'board' against what this changes for replay, cross-machine hubs, and any check that compares or re-renders board files, and against the entry's wish that the README never disagrees with what a run does."
---

Seed pass: the entries come from the board entry, the diff and the change description; none is a verdict.
