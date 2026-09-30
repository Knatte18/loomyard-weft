Read the closed GitHub issues #295, #277, #286 and #297 (all folded here), plus these findings that were never filed:

- **Status is unreadable.** `lyx loom status` gives raw JSON; `stuck` means both normal Bouncer-to-Burler routing and a real failure; nothing says how far the run has come (which segment of the recipe, what is left). The operator cannot follow a run without reading traces.
- **A deploy does not upgrade running sessions.** `lyx loom start` finds the existing status strand via `IfAbsent` and keeps its old display after `update-plugins.sh`.
- **Layout rule.** Every pane but the bottom-most collapses to a fixed configurable minimum (3 rows) and the bottom-most takes the rest; adding a pane moves nothing above it (`3+20` becomes `3+3+17`). Selvage is excluded.
- **ly-drive reporting.** Name the run by its slug, never `self`; give the run's own history position, not ly-drive's step counter; say when a reported state is from before it acted. `_lyx/shed/self/` should be named by slug, with `self` only an alias.

#286 replaces the old board entry `stencil-drift-remedy`.