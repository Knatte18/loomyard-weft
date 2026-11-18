---
round: 2
exclude_lenses: []
focus:
  - "D2 of decision-record.md: where the viewer ends after ensureRunsStrand replaces the overview loom-status strand with an own-window KeepViewer spec and then adds the runs strand. Reed facts to check: ReplaceStrand with a non-overview spec kills panes with selectOnEnd=true (internal/reedengine/replace.go), a viewer on an ending window moves to relevantWindow (internal/reedengine/ownwindow.go), a KeepViewer window is never selected, and AddStrand of an overview never selects its window (internal/reedengine/spawn.go). Settle whether the record names a decided step that selects the beam overview window, bounds it when the viewer was on another window, and names the test tier that pins it."
  - "D3 of decision-record.md: for a strand that declares a window group but is placed in a window of its own, settle which of the Title rule and the shared-group rule applies, and whether the D3 criterion names that case. Also settle whether D3 states why two own-window strands sharing a Title is harmless, given reed's window addressing by id or temporary name and OpenWindow's same-name lookup."
  - "D4 of decision-record.md and its criterion: settle whether the condition the criterion checks equals the condition D4 decides, for a worktree run that has a seed and a status file that is not done, including a halted or paused run."
---

Round 1 left one BLOCKING design finding on D2's viewer end state, with LOW findings on D3 and D4.
No lens is excluded: the walkthrough lens raised the viewer gap at MEDIUM in this round, so its area is not settled.
