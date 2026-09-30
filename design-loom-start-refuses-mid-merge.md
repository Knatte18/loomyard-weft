`lyx loom start` does not check for an in-progress merge.
A hub merge-in stopped on a conflict, and the following `loom start` spawned a driver onto the conflicted tree.
Preflight should refuse with the remedy: resolve, `lyx fabric merge-stage`, `lyx fabric merge --continue` (or `--abort`).