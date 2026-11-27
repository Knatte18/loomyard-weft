---
round: 1
focus:
  - "`latestDarnOutcome` in internal/loomcli/darnwiring.go against `renderPriorWork`: walk the history a run holds after an interrupted-stop halt and re-step, including an earlier recorded outcome followed by an interrupted spawn, and settle whether the note the next spawn gets is true about which spawn was interrupted and how the last one ended."
  - "The scratch listing: settle what can be under the `.scratch` link at the moment Darn-Write first spawns (earlier rows, the operator, gates, tooling, other agents of the task), and whether any of it makes a spawn with no predecessor be told one was interrupted."
  - "`listScratchFiles`, `fileExists` and `bulletList`: settle behaviour for a dangling link, an unreadable or vanishing entry, symlinks and nested trees inside the directory, a large tree, and unusual file names rendered into the prompt, and which of these halts the row. Also settle whether the order the cap cuts by keeps the items a resumed writer needs."
  - "The `BranchCommits` seam (`taskMergeBase`, `CommitLog`, `CommitsNotIn`): settle that the list holds only commits the note can truthfully attribute to an earlier spawn, including after a parent merge or on a fallback base, that its order is the newest-first the note states, and that a base that cannot be read (a run with no recorded parent) does not newly halt a spawn that worked before."
  - "contracts/stencils/darn/darn-template-write.md reads any prior work other than `(none)` as saying the branch commits are the writer's own to continue from: read it with the new interrupted note, including the scratch-files-only case, and settle whether the whole prompt stays consistent and accurate."
  - "Test coverage of the new wiring and behaviour: settle whether anything checks `darnDeps`' `BranchCommits` and `ScratchDir` values and `fabricengine.ScratchLinkPath`, and whether the lists following a recorded outcome are covered for scratch files as well as commits."
  - "_lym/landing/summary.md against the diff: check its design claim that rows before Darn-Write commit only to the records side, and its `## Merge danger` door and blast radius against every package the diff touches."
---

Seed pass: the entries name where round 1 should look; none states a verdict.
