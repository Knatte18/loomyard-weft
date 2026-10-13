# Card 7 — The create-worktree line drops its mutations field

**Edit:**
- `internal/battencli#battenCLI.wire`

**Intent:**
D10, second line: `battencli: create worktree` in the `CreateWorktree` closure `wire` builds drops its `mutations` field and keeps `slug`, since `fabric: mutation` already logs the same record from inside `Topology.Add`.
No test asserts the line or its fields (a grep of `internal/battencli` tests for `create worktree"` and `mutations` finds none), and a trace field no reader parses is no behavior a test pins, so the card adds none.

**ImpactSummary:** one field removed from one Info line in one closure; no caller, envelope or record changes.

**Commit:** 7: battencli: drop the duplicate mutations field from the create-worktree line
