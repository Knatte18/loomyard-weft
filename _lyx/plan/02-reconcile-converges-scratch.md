# Card 2 — reconcile-converges-scratch

**Create:**
- `plan:internal/fabricengine_test#TestReconcile_ScratchLink` -> `func TestReconcile_ScratchLink(t *testing.T)`

**Edit:**
- `internal/fabricengine#Topology.repairPairWiring`
- `internal/fabricengine#applyStaleRemoval`
- `plan:internal/fabricengine_test#TestWireJunctions_ScratchLinkRefusedAfterClassify`
- `internal/fabricengine_test#TestReconcile_RecreatedWeftIsWiredInTheSamePass`
- `internal/fabricengine_test#TestReconcile_MissingWeftRepoIsDiagnosedByName`
- `internal/fabricengine_test#TestReconcile_CorrectJunctionsAreNoOp`

**Prosa:**
- `internal/fabricengine/doc.go`

**Uses:**
- `plan:internal/fabricengine#wireScratchLink`
- `plan:internal/fabricengine#wireJunctionSet`
- `plan:internal/fabricengine#withoutRootScratchLink`
- `plan:internal/fabricengine#scratchOutcome.detail`
- `plan:internal/fabricengine_test#scratchShapeCases`
- `plan:internal/fabricengine#SetScratchBeforeLinkHookForTest`
- `internal/fabricengine_test#findReconcilePair`
- `internal/testkit/logcapture#Capture`

**Intent:**
`lyx fabric reconcile` converges the scratch link and exclude of every managed pair and reports what it changed or left in the pair's result (D2, D3).

`Topology.repairPairWiring`'s re-wire branch calls `wireJunctionSet` in place of `WireJunctionsWith`, so the step never runs twice in one pass.
After the junction repair block, on the healthy and the re-wired branch alike and whether or not the repair set `pr.Error`, it calls `wireScratchLink(rec, warpLayout, slug)` once and, when the outcome's `detail()` is non-empty, appends it through `appendPrDetail`.
The step changes no `Action` and sets no `Error`: a converted healthy pair stays `already_healthy` with the conversion in its Detail, because scratch is agent convenience and must never downgrade a verdict about the pair (D2).
`repairPairWiring` is reached only when the weft worktree existed at the start of the pair's iteration or reconcile recreated it in this pass without error, so the step never runs on an unmanaged worktree, a raw-adopted one, or a pair whose weft is still missing, and creates nothing for them; no guard of its own is added for that state, which `Reconcile`'s `repairWiring` condition already rules out.

`applyStaleRemoval` filters its scanned names through `withoutRootScratchLink`, so the stale-junction sweep never claims the root scratch link and never strips its `/.scratch` line (D3).
Without the filter the sweep's removal is refused by the gate's expected-target check, since `WarpJunctions` derives `<weft>/.scratch` while the link points at `<weft>/.lyx/scratch`, and `applyStaleRemoval` only logs that refusal through `logger.Warn` as `stale-junction removal refused`; that warn is the one observable difference, so the no-op row below captures the logger and asserts it is absent.

Tests:
`TestReconcile_ScratchLink`, in `scratchlink_integration_test.go`, runs parallel rows over a `hubforge.CopyHub(t, hubforge.Shape{Anchor: "."})` hub, each pair found with `findReconcilePair`:
- a junction-healthy prime pair whose root `.scratch` is a real directory with a file, its target absent, and whose warp exclude holds the legacy `/.scratch/` and not `/.scratch`: `Reconcile` leaves the pair `already_healthy` with no Error and a Detail naming the target, the record holds `path_renamed`, and the exclude holds `/.scratch` and no `/.scratch/`; the link and read-through card 1's row already pins are not asserted again;
- the same real `.scratch` shape on a prime pair whose `_lyx` junction was replaced by a real `_lyx` directory holding a file, which `seedLyxJunction` refuses: the pair carries a re-point Error, and its root `.scratch` is still a link to its target with the file read through it, since the step runs whether or not the repair failed;
- a raw warp worktree, added with `git worktree add -b <branch> <hub>/<slug>` from the prime warp with no weft branch and no `_lyx` at its anchor, whose root `.scratch` is absent: the pair is `raw_adopted`, and neither its root `.scratch` nor its target exists after `Reconcile`;
- the same worktree with a real `_lyx` directory at its anchor and its weft branch absent locally and on origin: the pair is `unmanaged_reported`, and neither its root `.scratch` nor its target exists;
- every `scratchShapeCases` row: `Reconcile` returns the pair with no Error and a Detail containing the row's report phrase, the root path (the exclude path for the failed exclude write) and `lyx fabric reconcile`; the filesystem outcome card 1's rows pin through `WireJunctionsWith` is not asserted again.
`TestWireJunctions_ScratchLinkRefusedAfterClassify` keeps its `WireJunctionsWith` rows and gains the same three shapes driven through `Topology.Reconcile`: the case-5 row's Detail names the target and `lyx fabric reconcile`; the case-6 rows' Details name the root and `lyx fabric reconcile`; after the hook-written-file case-6 row, a second `Reconcile` with the hook cleared reports the regular file left at the root as a case-4 leave.
`TestReconcile_CorrectJunctionsAreNoOp` turns serial, captures the logger with `logcapture.Capture` before its `Reconcile`, and adds to its `already_healthy` and empty-Detail assertions on the pair, whose root `.scratch` is the link `AddPairWith` wired: the link still points at its target, the exclude still holds `/.scratch`, the record holds no `link_removed`, and the captured output holds no `stale-junction removal refused`, which the unfiltered sweep logs for the link.
`TestReconcile_RecreatedWeftIsWiredInTheSamePass` removes the pair's root `.scratch` link before the weft removal and asserts the pass that recreates the weft also leaves `.scratch` a link to its target, with the target present.
`TestReconcile_MissingWeftRepoIsDiagnosedByName` removes the prime's root `.scratch` link and its target before reconciling, and asserts neither exists afterwards: the pair reported `unmanaged_reported` with its weft missing gets no link and no target.

Docs: the `# The scratch link` section of `internal/fabricengine/doc.go` states that reconcile runs the step on every managed pair after the junction repair, reports its outcome in the pair's Detail without changing Action or Error, skips pairs that never reach the repair, that the stale sweep never claims the link, and that after this lands the operator runs `lyx fabric reconcile` once to convert the prime and every pair wired by an older binary.

**ImpactSummary:** `repairPairWiring` and `applyStaleRemoval` are reconcile-internal with one caller each; pair Actions are unchanged and Details gain a scratch note only when the step changed or left something, which the existing no-op test's empty-Detail assertion still holds for a converged pair.
