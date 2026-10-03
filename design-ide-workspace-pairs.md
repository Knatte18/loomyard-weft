# Pairs as folders in the prime's hub workspace

`ide-workspace-pairs` · Someday · design

Idea, undecided (operator, 2026-10-03): keep the prime's hub workspace as the one VS Code window, and have fabric add each pair to its `folders` on `add` and drop it on `remove`/teardown; VS Code picks up workspace-file edits live.
The operator rarely reads a pair's files (everything goes through the orch) but looks at plan files and similar now and then, which `_portals` (each pair's `_lyx/`) already covers and is kept for.
Open points: per-folder `folderOpen` tasks would run in the prime window (and batten's own pair window would then go), a full pair copy multiplies indexing and search unless only `_lyx/` is added or excludes are set, and the prime must stay the first folder or the extension host restarts.
