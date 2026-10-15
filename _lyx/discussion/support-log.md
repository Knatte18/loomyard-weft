# Support log — twb-d0-gc

## Interview

### Turn 1 — exploration (autonomous; no operator, no advisors)

Read the board entry and `plan-drift-refusal-route`, `weaver`, `glyph-use-audit` for overlap.
Read `postBatchChecks` (`internal/websterengine/recordbatch.go`), `DoneChecks` (`internal/planglyph/donecheck.go`), `BindHandles` (`internal/planglyph/handle.go`), `gateScopeOf`/`renderBatchGate` (`internal/websterengine/cardgate.go`), `CardTargetDirs` (`internal/planparser/targetdirs.go`), the batcher package doc and `partition.go`/`rebaseline.go`.
Found: `DoneChecks` resolves against the worktree after the batch; `BindHandles` matches handles against the batch's net delta, so a handle into a package a later card renames finds no `Created` entry; `renderBatchGate` unions directories at `begin-batch`.
Found: `loom-plan-spec.md` already states "A card that renames or deletes what an earlier card targets never shares that card's batch", and the cost batchifier does not implement it.
Ran `lyx gate test ./cmd/lyx`: 8 s wall, 3 s test time.

### Turn 2 — batch of questions, each auto-picked

1. Item 1, how to stop the false rejection? Recommended: implement the spec's batching rule in the cost batchifier. Alternatives: resolve per card commit; follow plan renames in the checks. Picked the recommendation.
2. Item 2, how does the batch gate survive a rename? Recommended: the same batching rule, so the batch's final tree holds every package its gate names. Alternative: drop removed directories from the batch gate's union. Picked the recommendation.
3. Item 3, plan time or batch time? Recommended: batch time, a configured list of always-run packages. Alternatives: flag pinned symbols in `validate-plan`; run `./cmd/lyx` in every card gate. Picked the recommendation.
4. Re-check recorded partitions against the rule? Recommended: no. Picked no.
5. Tagged or untagged extra step? Recommended: untagged. Picked untagged.

### Turn 3 — lens review by forks

Read card 12 of the tender plan from the weft's `archive/tender/b367e77fb6c1` tag: a `**Rename:**` group of file pairs from `internal/pausecli` and `internal/pauseengine`.
Every fork returned a report.

- board-fit, MEDIUM, D5 never runs on an alignment-approve landing: fixed as proposed; Open risks states that the operator runs it on that route.
- board-fit, MEDIUM, D5 depends on `config-set-validate` for a list `--set`: fixed as proposed; D5 names the exact command, a read-back, and that the template ships `[]` so `--set` takes a flow list; 512 concerns a model value, and D5's load refusal catches a mangled value.
- board-fit, LOW, "neutral to the hub" overstated: fixed as proposed; D5 says the key is for a Go hub and drops the neutrality claim.
- board-fit, LOW, `weaver` unnamed: fixed as proposed; D5 says any spawn path built on `renderBatchGate`, `weaver`'s included, carries the step.
- board-fit, LOW, bench-arm siblings `twb-d0-ga`/`twb-d0-gf`: fixed differently; the finding proposed no record change, and the ledger notes them as bench arms, the operator's matter.
- goal-scope, MEDIUM, card 12's labels unestablished: fixed as proposed; D1 cites card 12's `Rename` file pairs, and the predicate became overlap in either direction so file pairs cover the package-scoped handles.
- goal-scope, MEDIUM, D5 outside the task with no criterion: fixed as proposed; acceptance row for the PR description's command, command named.
- goal-scope, LOW, `Move` unlicensed by the spec line: fixed as proposed; Scope adds the `loom-plan-spec.md` line gaining "moves".
- goal-scope, LOW, accept-verb boundary: fixed as proposed; Scope Out says D1 removes the known cause and other causes still need a fresh restart.
- goal-scope, LOW, Goal promises `./cmd/lyx`: fixed as proposed; Goal no longer names the hub value.
- leanness, MEDIUM, auto-mode bullets point at the support log: fixed as proposed; cut.
- leanness, MEDIUM, 8 s figure twice: fixed as proposed; kept once in D5's rationale.
- leanness, MEDIUM, D5 not buildable and repeated in Open risks: fixed as proposed; folded into the batch-gate decision as one paragraph, the gap kept once in Open risks.
- leanness, LOW, Goal restates Scope: fixed as proposed; one sentence.
- leanness, LOW, Constraints repeat D2/D4/Scope: fixed as proposed; the chokepoint and docs lines cut, the others name only what Decisions do not.
- leanness, LOW, Scope Out reasons: fixed as proposed; reasons cut but for the accept-verb line, which goal-scope asked for.
- leanness, LOW, Notes duplicates: fixed as proposed; `postBatchChecks` and test-budget bullets cut.
- leanness, LOW, D1 rationale versus Notes: fixed as proposed; covered by cutting the Notes bullet.
- premises, MEDIUM, a package `Move` still names its source in its own gates: fixed as proposed; new D3 drops a whole-file or package `Move` source from gate scope, with acceptance rows.
- premises, LOW, "forced to start a batch" imprecise: fixed as proposed; D1's Bound states the pairwise rule.
- premises, LOW, the cost DP grows segments backward: fixed as proposed; Notes say so and name the `break`.
- premises, LOW, name the command: fixed as proposed, with the board-fit finding.
- premises, LOW, "quarry resolves a worktree, not a commit" unchecked: fixed as proposed; the rejected alternative now rests on the missing per-card SHA trail alone.
- scenarios, MEDIUM, a listed package a card of the batch removes: fixed as proposed; D5 drops it by D2's predicate, acceptance row added.
- scenarios, MEDIUM, `./` check too weak: fixed as proposed; entry shape refuses `...`, `..`/`.` segments, whitespace, a leading `-`, each a row.
- scenarios, MEDIUM, in-flight runs keep the bug: fixed as proposed; Open risks names the fresh restart.
- scenarios, LOW, rebaseline tail row: fixed as proposed.
- scenarios, LOW, one-card segment admissible: fixed as proposed; D1 states it, row added.
- scenarios, LOW, `Rename` to-side handle unit: fixed as proposed; D2 states it, row added.
- scenarios, LOW, a layer without the key: fixed as proposed; row added.
- pattern-fit, MEDIUM, refusal row and test: fixed as proposed; D5 names the extended "config file content invalid" row, its way forward, and Constraints a test reaching it; Scope lists the `refusal-spec.md` edit.
- pattern-fit, LOW, name the command: fixed as proposed, merged with the board-fit finding.
- pattern-fit, LOW, D3 reasoning unlike `CheckBatchOrder`: fixed as proposed; D4 (was D3) says D1 is pinned by the cost batchifier's tests.
- pattern-fit, LOW, containment computation: fixed as proposed; D2 compares `RefScope` outputs only.

## Rejected alternatives

- **Resolve each card's glyphs at its own commit.** Webster records one SHA per batch (`CardSHAs = []string{report.HeadSHA}`), not per card; this would need per-card commit attribution from the fork and a resolve per card commit, for a case the batching rule removes.
- **Follow renames in `DoneChecks` and `BindHandles`.** Mapping an earlier card's handle through a later card's `Rename` pair needs unit-prefix substitution across the glyph chokepoint and changes what a handle binds to; two checks change where one batching rule suffices.
- **Plan side: `validate-plan` flags a rename card earlier cards depend on.** Batching is "never the plan's decision" (`internal/batcher` doc); refusing a correct plan to work around a batcher gap is backwards.
- **Drop removed directories from the batch gate only.** Fixes item 2 and leaves item 1.
- **Flag tests that pin a symbol by name at plan time.** No mechanical map from symbols to the tests that pin them exists, and it would miss repo-wide scans such as tier purity.
- **Run `./cmd/lyx` in every card gate.** Catches at the causing card, but multiplies an 8 s step by the card count; the batch gate runs within the same fork, which fixes before reporting.
- **Hard-code `./cmd/lyx` in webster.** Webster serves other hubs, C# ones included.
- **Re-check a recorded partition.** Would wedge in-flight runs formed before the rule.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 item 1 mechanism — resolved, auto-pick: cost batchifier enforces the spec's rename/move/delete rule (D1).
- Q2 item 2 mechanism — resolved, auto-pick: D1, plus D3 for a `Move` card's own gates.
- Q3 item 3 placement — resolved, auto-pick: `batch_gate_always` in `webster.yaml`, untagged step (D5).
- Q4 recorded partitions — resolved, auto-pick: not re-checked (D4).
- Q5 where the predicate lives — resolved, auto-pick: `internal/planparser` (D2).
- Q6 this hub's value — resolved, auto-pick: `./cmd/lyx`, set on the board's repo default after landing (D5).
- Q7 manual accept verb — resolved, auto-pick: out of scope; the board's scope list does not ask for it.
- Q8 sibling entries `twb-d0-ga` and `twb-d0-gf` carry this entry's body — resolved, auto-pick: bench arms of the same task, left to the operator.
