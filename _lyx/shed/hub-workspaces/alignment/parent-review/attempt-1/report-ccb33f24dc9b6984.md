---
verdict: APPROVED
---

## Scope check

The design does what the `hub-workspaces` board entry asks, no more and no less.

- One workspace per umbrella, with only the prime today: D1 writes the root umbrella's file, and Scope Out leaves the other umbrellas' files to the task that builds them, under the naming the operator already chose (`<RepoName>-<umbrella-slug>.code-workspace`).
  That matches `orch-any-worktree`, whose umbrella Create step ("its pair, thread and workspace") owns those files.
- The file sits directly in the hub directory and carries the repository name (`loomyard.code-workspace`): D1.
- The folders are the umbrella worktree first, then Bolt, then every pair at its anchor directory, in alphabetical order, named `main`, `Bolt` and the slug: D2.
- `_portals` is retired: D6, with D7 sweeping existing hubs, which follows from retiring it on hubs that already hold it.
- A finished pair drops out: D4 regenerates the file on `Remove` and `Prune`, along with `Add`, `Clone` and `Reconcile`, which is the orch recommendation the entry records.
- Listing `--local` pairs (D2) is consistent with `fabric-remote-async`, which keeps local pairs only as an explicit operator choice.
- Batch worktrees are left to `parallel-batch-parts` and `weaver` and duplicate neither.
  Beams and dobby (`umbrella-daemon`) and per-umbrella anchors (`umbrella-anchor`) are not touched.

No goal point is left out, and no decision belongs to another entry or contradicts one.

## Overlaps

- Peer `implementer-seam` (row 0, unknown order against own row 3): its decision record names no fabric, workspace, portal, VS Code or ide surface, and it has no changed paths yet.
  It has no overlap with this run.
- Parent commit `c03af73` (plandag dev tool under `tools/plandag/`): no overlap with this run.

No overlap exists, so no landing order applies.
