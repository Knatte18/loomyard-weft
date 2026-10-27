# Board tidy counts landings per machine, not per owner

`tidy-count-per-owner` · Note · bug, workspaces

## Seen (operator, 2026-10-27)

OSL-1046 owned root and landed the runs there, and that machine's dobby ran board tidies over them.
When this machine pulled their run records, Neve's dobby, which did not own root, spawned a tidy of its own, its line reading "board tidy after 13 landings".
Each dobby counts the landings it reads into its own machine-local state record, so a tidy on another machine never resets the count here.

## Idea

- A dobby counts and tidies only the landings of a root or beam its machine owns.
- A tidy on any machine marks its landings as tidied for every machine, so an adopt does not tidy them again.
