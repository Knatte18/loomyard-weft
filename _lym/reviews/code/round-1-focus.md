---
round: 1
focus:
  - "internal/fabriccli/roleguard.go benchPairVerb: settle whether the slug form alone keeps a bench writer off another task's pair on both add and remove, including a board entry or batten run whose slug reads as <slug>-b<N>, given that the teardown checks for one and the guard does not. Check the answer key's 'a bench never touches another task's pair' bound."
  - "benchPairVerb and the `remove` and `add` commands: read every flag and positional form those commands accept, and settle whether any of them lets an admitted call do more than create or remove that one bench pair forked from the task branch, or destroy uncommitted or unpushed bench work."
  - "isBenchPairPath and injectedPathsAreOwn: the push admission and the verb admission apply different conditions (recipe, profile, calling directory). Settle whether the push side stays within the answer key's bounds, and whether the detached children that add and remove actually spawn all pass the guard."
  - "Worktree-Teardown coverage of 'none outlive the run': trace every path that ends a run or removes its task pair (internal/battencli/wire.go Remove, the session-shutdown half, an already-gone task pair, a retried or resumed row, abandon, handover, merged-elsewhere and any other pairteardown caller) and settle which of them reach RemoveBenchPairs."
  - "pairteardown.RemoveBenchPairs in internal/pairteardown/leftover.go: check it against RemoveOrphanPair for what it skips or changes (prunable or half-removed pairs, records sibling left alone, a pair whose Resolve fails, lock handling under the prime lock, ending live processes with Force and Verdict EntryEnded, a bench pair that holds its own batten run or child pairs). Settle whether a failure or partial state can strand a pair, lose data, or halt a row that a re-run cannot clear."
  - "Scope and docs: search the repository's stencils, specs, module docs, help text and any constraints document for statements that a bench or a task session cannot add or remove pairs, must ask the orch, or that list the guard's admitted verbs or teardown's steps, and settle whether the diff leaves any of them stale or leaves the writer with no way to learn of the new route before being refused."
  - "_lym/landing/summary.md: check each claim against the diff and tests, including what the Tests section says is pinned and the Merge danger section's door and blast radius against everything the diff can break (guard admission as well as teardown, remote archive tags and branches)."
  - "Test coverage: settle whether the Remove wiring in internal/battencli/wire.go (ordering and halt on failure) and the batten-run-directory skip in RemoveBenchPairs are pinned by any test."
---

Seed focus for round 1, drawn from the decision record's board entry and the diff over the review range.
The entry leaves a cap on the number of bench pairs open, so these entries do not ask for one.
