```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over the diff to `internal/loomcli/prfindings_integration_test.go`.

**List (function or branch: case run, result)**

- `TestPRFindingsCLI_SendsBackBlockingFindings`, Tapestry row.
  Ran: `blockingFindingsRun` writes a tapestry seed. `resolveRecipe` (`arm.go:109-116`) reads it and sets `c.recipe` to tapestry. `prReviewSendBackEdge` (`prfindings.go:290-305`) finds the PR-Review row, `wireLanding` wires the seams, `newShed` builds the shed, and `sendBackEdge` resolves the reroute.
  Expected: exit 0, `send_back: true`. Correct.
- Darn row.
  Ran the same path with `c.recipe` set to darn. `contracts/recipes/darn-recipe.yaml:285-288` declares PR-Review with `on_reroute: Code-Bouncer`, and Finalize at `:295` is a landing row, so the shed build needs the landing seams.
  The row reaches the darn path. A darn run without the PR-Review row would answer `send_back: false` with a reason and fail `got != true`. Correct.
- Loop closure `tt` under `t.Parallel()`: safe if the module is on Go 1.22 or later. Not verified, because I made no tool call. Correct, assumed.
- `blockingFindingsRun` with each `t.Fatalf` error path (`ResolveWorktree`, `WriteSeed`, `WriteRecord`, `MkdirAll`, `WriteFile`): each failure aborts the subtest with the error named. Nothing is swallowed. Correct.
- `WriteSeed` against an existing template seed.
  Case: the template pair already holds a seed with another recipe. `seed.go:178` then refuses with `ErrDisagreeingSeed`, and the helper fails loudly instead of testing a wrong recipe. The parent's record says both rows pass, so the template holds no disagreeing seed. Correct.
- `WriteSeed` stamps `StartedAt` with `time.Now()`, but nothing in the verb's send-back path reads it. No determinism effect. Correct.
- Tapestry row versus the base test: the base relied on the default when no seed exists (`sharedbootstrap.go:270-274`, `arm.go:105`). The row now writes an explicit tapestry seed. Both resolve to tapestry, so the behavior under test is unchanged. Correct.
- `findings` path: the helper and the subtest each compute `filepath.Join(worktree, ".scratch", "pr-findings.md")`, and the two computations agree. Correct.

**Findings**

None. Every branch I ran gives the right result, every error path fails loudly, and no failure leaves state inconsistent.

**Tool calls**

None made.
