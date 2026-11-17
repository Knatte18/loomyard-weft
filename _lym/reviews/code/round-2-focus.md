---
round: 2
focus:
  - "internal/boardcli/cli.go list command: does `list --text --label <x>` leave out the P0 row, and does a test in internal/boardcli/verbs_test.go assert the exact stdout of that run?"
  - "internal/boardcli/text.go withRootProjectRow: with open tasks plus done entries and no open project or note, does the P0 row land before the done entries, as the board layer's render order puts projects ahead of done entries, and does the doc comment match?"
  - "internal/boardcli/cli.go list command under --sort age or --sort revised: where does the P0 row land, and does that agree with the sort's rule that undated entries come last and with the list help text?"
  - "internal/boardcli/verbs_test.go stepProjects doc comment: does it describe the parent_beam value the step's assertions expect for a grouping project under the prime?"
---
