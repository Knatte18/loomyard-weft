```yaml
lens: fix-proposal
outcome: reported
```

Two findings, one per proposal.

**F1 — `delete(r.pairs, slug)` outside Run-Shed (`internal/battencli/overview.go`, `poll`)**

- **LOW:** The proposal drops the cached pair but leaves `childRecipes[slug]`, which is read through the same pair.
  - **Case:** a run of the pair is seen in Run-Shed, so `resolveChildRecipe` caches the recipe read at the old anchor. The run ends, the anchor moves, and a new run starts in Run-Shed.
  - **Result:** `resolveChildRecipe` returns the cached recipe at its first line and never re-resolves the pair or reads the seed at the new anchor. A different recipe in the new run's seed would be mislabelled in `ChildRecipe`, so the child row's stage number would be wrong.
  - **Evidence:** the `resolveChildRecipe` cache hit returns before `r.childPair(slug)`. The `childRecipes` comment says a seed is write-once, which no longer holds across a pair move.
  - **Test gap:** the new test writes no child seed, so it does not exercise this.
  - **Fix:** `poll` should also `delete(r.childRecipes, slug)` in the same `else` branch.
- **LOW:** The second Run-Shed read in the new test only passes because the child status file got a distinct modification time.
  - **Evidence:** the first test run failed with `Stage: "Plan-Write"` until `writeChildStatus` took a distinct `modTime`. `StatusCache.Poll` keys on modification time alone and ignores the path.
  - **Why it matters:** `childCaches[slug]` is not dropped with the pair. A moved directory keeps its file times, so a child status at the new anchor with an equal modification time would still show the old content.
  - **Likelihood:** a new run normally writes a fresh status, so I rate this low.

**F2 — `commandOf` seam plus a tier1 test running `status`**

- **LOW:** The test runs the real `status` verb to trigger the pre-run, so it also runs `status`'s `RunE` against real reed.
  - **Evidence:** `status.go` documents that a live strand with a dead watcher gets a new watcher spawned. That is a side effect on the forged hub, and the test ignores `status`'s result.
  - **Why it is low:** the test passed and the hub is a temp copy, so this is fragility and not a defect.
  - **Alternative:** call `commandOf(c).PersistentPreRunE` directly, or run a verb whose `RunE` is inert.

Tool calls:
1. `grep` of the `statusCmd` section in `internal/orchcli/*.go`: my context did not show `status`'s `RunE` side effects.
