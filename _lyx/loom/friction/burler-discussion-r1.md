# Burler fix, discussion round 1 — live hub state ahead of HEAD

F6's proposed replacement sentence, "loom and darn runs are all seeded llm", was false: live loom seeds across the hub record `"driver": "go"`, this task's own included.
The hub's `_board/_lyx/config/loom.yaml` already holds the override shape this record designs (empty values inheriting `shed.yaml`).
The reviewer and the record both reasoned from HEAD's template, and the fix could only be checked by reading live pair records outside the worktree.
Telling HEAD's code from the deployed build's state cost real time.
