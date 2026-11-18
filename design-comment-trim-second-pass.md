# Comments: check what the big comment rewrite left, before any new pass

`comment-trim-second-pass` · Task · enhancement, docs · running · Record-Write

## What was seen

Last week a large run (nearly 3,000 agents in parallel) rewrote and trimmed docstrings and doc files across lym.
Even so, the shrink study (2026-11-18) still found comments at 20 to 31 % of production lines in the seven most patched modules,
and in `landingshed` a 678-line `doc.go` that mostly restates the function docs.

## Idea

Before any new comment pass, check whether that rewrite covered package `doc.go` files and how long they still are,
and list the packages whose comment share stands out. Only then decide whether a narrow second pass (for example `doc.go` only) is worth it.
