# A batten pass after a landed child reports a false rewind against origin

`false-rewind-after-finalize` · Note · bug, batten, fabric

## Seen (orch, 2026-11-01)

Twice today a batten pass after a done child stopped with "branch <slug> is rewound against origin", first `heddle-orch-target`, then `batching-in-claude-local`.
Both times origin's task branch equalled the local one (`batching-in-claude-local` at 792a340a), and the commit the stop called origin's rewound head (31119e4e) was an ancestor of it.
The dropped list held commits of main that Finalize had merged into the task branch.
The orch re-ran `lym batten run <slug>` from the prime without touching git, and both runs landed.

## Idea

The rewind check reads origin's head from a stale ref, likely one read before the push Finalize made, or during a concurrent fetch of the same ref.
It re-reads origin's head after a fresh fetch, and calls a branch rewound only when the local head is not an ancestor of origin's head.
It may share its cause with `reconcile-fetch-race`; check that fix first.
