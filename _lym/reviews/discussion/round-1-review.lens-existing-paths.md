```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths, the code paths that already touch the state D1–D9 introduce or change.

**Paths traced, per decision**

- **D1/D2, `cmd/testtiming` report and `-pkg` splice (`runRedundancy`, `spliceReport`):** survives. The engine move leaves report writing and splicing in `main`.
- **D2, the `coverpkg` list built from `layout.dirs` (redundancy.go:378-382):** survives. A module-wide binary still holds the package's own blocks.
- **D4, map record via `internal/state` under the hub's `.lym`:** survives. I found no cleanup path that clears hub `.lym` files (grep of fabricengine RemoveAll sites turned up only integration fixtures).
- **D5, the prime worktree that the verb runs in vs Finalize's landing:** breaks (F2).
- **D6, the prime dobby's tick vs the board-tidy landing count (`BoardTidyEvery`, config.go:36):** survives. D6 is independent of the count.
- **D6, the dobby restart or stop path vs an in-flight `lym gate testmap` child:** breaks (F4).
- **D7, the gate key's `reports` list vs the hub base file's `keys` overrides:** breaks (F1).
- **D7 step 1, "the gate's existing diff route" (`cardChanges` to `impactset.ChangedPaths`, scope.go:476-495):** breaks (F3).
- **D7, `parentMergeBase` in a worktree with no parent branch (scope.go:668-677: "such as the prime on main, reports false"):** the record is silent on what step 1 does then (F3).
- **D7, the green-record carry (green.go, `greenEntry`):** survives for the gate result. The step reruns its whole budget on every carried call over an unchanged tree (F5).
- **D7, `parent-align` key readers (`landingshed/doc.go:354`, "It runs its own gate key, `parent-align`, over the merged tree"):** survives. The key exists and runs over the merged tree.
- **D8, the report in the worktree's gate directory vs later gate calls:** survives. Each later key's call rewrites the report, and D8's content match decides whether it is current.

**Findings**

- **F1.** Severity MEDIUM, class design, location D7, last sentence ("The shipped template lists the step on the `webster`, `parent-align`, `darn-write` and `code-burler-fix` keys").
  The template change never reaches a hub that already tunes those keys, including this one, so the step never runs and the goal, stopping redundant tests from piling up, is not met here.
  Evidence: `_bolt/_lym/config/gate.yaml:46-51` holds `webster`, `parent-align`, `darn-write` and `code-burler-fix` whole, with tiers `[0, 1, 2]`. `PATTERN-config-layers` says the hub base file holds tuned units and wins over the template.
  The record should say how such a hub's keys get `reports` (an operator `lym config gate --set`, or a reconcile step) or state that it stays manual.

- **F2.** Severity MEDIUM, class design, location D5/D6 (the prime-worktree refresh).
  A Finalize landing into the prime fast-forwards the prime's checkout in place while `lym gate testmap` is running tests there.
  A package's tests then run on a mix of old and new files, while the map records the commit and file hashes of one tree.
  D5 checks dirtiness only at the start ("skips a package ... that differs from HEAD") and holds no lock against the landing.
  Evidence: `internal/fabricengine/merge.go:413-418` takes the weft write lock and runs `git merge --ff-only` in the weft checkout. D5 names only its per-package lock through `internal/lock`.
  The record should state whether the verb takes the weft write lock or re-checks HEAD and file hashes after each package, discarding the map on a change.

- **F3.** Severity MEDIUM, class design, location D7 step 1 ("read through the gate's existing diff route").
  The existing route returns changed paths only and never the base-side text, so it cannot tell which test functions differ from the merge base.
  Evidence: `internal/gatecli/scope.go:476-495`, where `cardChanges` returns `impactset.ChangedPaths(ctx, root, base)` (paths).
  Reading a function's text at the merge base needs a git read at a revision, which `PATTERN-fabric-boundary` keeps to `fabricengine`. The only file-at-revision call seen there is `AnchoredFileAt` (trailercommit.go:151), and the record does not name it or say whether it serves the weft.
  Also, `parentMergeBase` reports false for a worktree with no parent branch (scope.go:668-677), and D7 does not say what step 1 judges then.
  This is a fact the design rests on, left for Plan.

- **F4.** Severity LOW, class design, location D6 ("waited on by the dobby", "no refresh is in flight").
  "In flight" lives in the dobby's memory. A dobby that stops or restarts mid-refresh leaves the child holding a gate slot and package locks, with no owner record and no ender.
  The process is long-lived and belongs to the hub, and the record does not register it under `runproc.HubOwner` or name what ends it.
  Evidence: `pattern/PATTERN-child-processes.md`, "Run process owner": a hub process "sits under `runproc.HubOwner`", and processes are ended only through `End`, `EndSparing` or `EndRole`.

- **F5.** Severity LOW, class design, location D7 ("after the key's tiers, carried or run").
  On a `lym gate test` call whose tiers are all carried from the green record over an unchanged tree, the step still rebuilds and reruns up to `redundancy_budget_min` of tests, giving the same report.
  D7's cost rationale ("one coverage build per touched package ...") counts one run, not one per call.
  Evidence: `internal/gatecli/green.go:1-2` (the green record that carries a passed tier forward) and D7's step placement.
  The record should state the repeat cost as a bound, or skip the step when the existing report matches the tree, as D8's match already defines.

**Tool calls**

1. Grepped `_bolt/_lym/config/gate.yaml` for landing/final/round gates and lymdirs/gatecli for a gate-dir resolver. The manifest lacked which call each verify gate renders.
2. Grepped fabricengine, lymcleanup and tendercli for `RemoveAll` and lymdirs for the gate directory. The manifest lacked cleanup paths over hub `.lym` and the gate dir.
3. Grepped internal for gate-dir functions and removers, and for `parent-align` readers. The manifest lacked who runs the `parent-align` key and whether anything clears the gate dir.
