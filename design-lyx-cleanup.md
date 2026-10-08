# lyx cleanup: one command runs every module cleanup

`lyx-cleanup` · Note · feature, batten, board

## Wanted

The operator (2026-10-08): one command that tidies the repo, running every cleanup there is.
Today finished batten runs leave `_lyx/shed/<slug>/` behind in the records repo for good (one directory per slug that ever ran), done board entries wait for `lyx board prune`, and `.scratch/` and `.lyx/` gather drafts and logs nobody reads again.

## Proposal

- A top-level verb, such as `lyx cleanup`, that only calls each module's own cleanup verb, so no module learns about another's state.
- Batten's cleanup removes the status directory of every run whose status is terminal and done; it never touches a running, halted or failed run.
- It runs `lyx board prune`.
- It clears what `.scratch/` and `.lyx/` hold that can be discarded: old orch notes beyond the latest, logs past an age, stale drafts.
  Which `.lyx/` paths are disposable is a design call for the task that builds it.
- A dry run lists what would go before anything is removed.

Related: note `batten-reused-slug`, where a leftover done status blocked a new task with the same slug.
