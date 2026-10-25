```yaml
lens: board-fit
outcome: reported
```

Board-fit lens for the regression-baseline-verify decision record: checking its fit with the board, its peers and parent commits.

Findings:

1. `decision`, MEDIUM. Open risks / D2. The board entry is outdated by landed work, so this goes to the parent.
   Question for the parent: should `Drift-Verify` sit after the Code perch and before Describe, which compares a tree that carries parent commits merged by `Parent-Align`? Or should it sit between Webster and `Parent-Align`, which leaves the Code perch's fixes uncompared?
   Evidence: since `fc75aef79` (2026-10-24) the loom recipe runs `Webster → Parent-Align → Code-Bouncer → Describe`. The entry says "after Webster and the Code perch, before Describe and Parent-Align", and both halves can no longer hold.
   The record already raises this under Open risks, so this finding only confirms it goes to the parent.

2. LOW. D1 (`capture.dir` and `compare.dir` "relative to the worktree root") and D4/D5 (`.lyx/drift/` "under the worktree").
   The record does not say which root a hub with a non-root anchor resolves against.
   The peer run `umbrella-anchor` (at Plan-Write) gives each worktree its own anchor, and its D3 builds every lyx path for a worktree from that worktree's anchor.
   Models' project root is a subdirectory (`src/csharp/NORCE.Models/`), which is exactly the case where anchor and git root differ.
   The fix: state that `dir` resolves as gate tiers' `dir` does, and that `.lyx/drift/` and `_lyx/drift/` sit wherever `internal/lyxdirs` puts `.lyx` and `_lyx` for the worktree.
   This is not a conflict: the peer's `lyx fabric anchor` refuses an unfinished run, so an anchor never moves under a live capture.

3. LOW. Scope "Out" and the testing line ("nothing here needs Models", "an end-to-end run, which follow on OSL-1046").
   The record does not mention the C# sandbox hub that already exists on Neve.
   The `csharp-sandbox-hub` note, a member of the same `dotnet` project, records the hub at `/home/knatte/Code/lyx-csharp-LYXHUB`, with NUnit and gate tiers set up. Its stated purpose is end-to-end C# runs on Neve before OSL-1046.
   The record can cite it as the place where a Drift-enabled end-to-end check can run without Models. It must not file or promote anything for this.

What I checked:
- Board entries:
  - `umbrella-anchor` (running peer): overlap on path resolution under an anchor (finding 2), no conflict.
  - `csharp-sandbox-hub` (note, same project): overlap (finding 3), no conflict.
  - `dotnet` (project P2): its order lists this task first, then `csharp-sandbox-hub`; no conflict.
  - `codex-astra-solo-bench`, `beam-promote-keeps-discussion`, `run-verbs-shared` (`lyx loom goto` on darn runs, which the record's way-forward text uses): no conflict. `run-verbs-shared` is a note about a future move only.
  - The rest of the board list: no overlap.
- Peers: `umbrella-anchor`, as above.
- Parent commits: `lyx loom peers` lists none. `fc75aef79` comes from the writer's exploration, behind finding 1.
- Decision notes: the board list holds no note titled `Decision for regression-baseline-verify:`, so there was none to read.

Tool calls:
- `lyx loom peers`: my context lacked the peer and parent-commit list.
- `lyx loom peers show umbrella-anchor --record decision`: my context lacked the peer's decisions.
- A grep of that saved output for anchor-relative paths: the output was too large to read whole.
- `lyx board get` for `csharp-sandbox-hub` and `dotnet`: my context held only their titles and briefs.
