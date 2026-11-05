---
format: 5
approved: false
language: go
---

# Plan: drift-rework

Drift stops depending on an agent's reading of an opaque compare.
Go diffs the baseline capture against a current capture into changed values, checks them against an expectation the expect agent writes and Drift-Expect seals before the writer starts, and a judge rules on each expected value against its exact old and new value, stored as exact tuples.
The plan builds the general fabric seams first, then the pure diff, config and parsers, then the guarded records, the seal and the current capture, the shared containment check with its gate entry, the Drift-Expect row, the `lym drift check` verb, the judge seat, the rewritten Drift-Verify, the code-only Drift-Baseline, the parent's amendment, the verdict and the writers' stencils, and removes the old machinery last, once nothing reads it.

## Card Index

1 — fabric-trailer-commits — commit weft paths with extra trailers, find the oldest or newest commit carrying a trailer, read a file at a weft commit
2 — fabric-checkout-states — read both checkouts' HEAD, changed tracked paths and untracked paths
3 — drift-capture-diff — diff two capture directories into keyed and unkeyed cells
4 — drift-config-keys — add diff and key_patterns, rename the model keys, refuse hub-only keys in run_settings, flag the meaning change
5 — drift-expectation-parser — the sole parser of expected.yaml and its coverage of cells
6 — drift-judgement-parser — the sole parser of judgement.yaml, and the PATTERN-sole-parsers line
7 — drift-records-guard — scope.json's new fields, judged.json, and the Drift-Records commit guard
8 — drift-seal-baseline-checks — the Drift-Seal check, the baseline digest and the run's row order
9 — drift-current-capture — slot before lock, and a scratch current capture that waits for the lock
10 — drift-containment-check — the shared check and the drift gate entry on Darn-Write and Webster
11 — webster-runlevel-entry-report — a failed run-level entry reaches Merriam's verify fixer through the verify-gate report
12 — drift-expect-producer — the Drift-Expect engine, its expect seat, stencil and expectation entry
13 — drift-expect-rows — Drift-Expect joins the tapestry and darn recipes
14 — drift-check-verb — lym drift check, the drift gate's check on demand
15 — drift-judge-seat — the judge seat, stencil, judgement entry and the snapshot guard
16 — drift-verify-lines — Drift-Verify as containment and judge under lines mode, and Drift-Recheck on the same engine
17 — drift-verify-command — the command mode's compare and per-case judge
18 — drift-baseline-await — Drift-Baseline records the scope from config with no agent, Drift-Await records the baseline digest
19 — drift-amend-defer — lym drift amend and the wider lym drift defer, with the parent caller check
20 — drift-verdict — the verdict rendered from the committed records
21 — drift-writer-stencils — the drift paragraph in the darn writer's, Merriam's and the card fork's stencils
22 — driftcli-retire-verbs — remove scope, accept, withdraw, waive, compare and capture --side
23 — drift-retire-recheck-engine — remove the DriftRecheck engine and constructor
24 — drift-retire-old-seats — remove the baseline and verify agents' seats, stencils and seams
25 — driftengine-retire-acceptance — remove acceptances, the run budget and side capture
26 — drift-retire-skip-keys — retire skip, verify_max_runs and {skip_file}
27 — drift-retire-verify-gate-key — remove the drift-verify gate key

## Shared Decisions

### Decision: trailer-commit-adds-warp-sha

- **Decision:** `CommitAnchoredPathsWithTrailers` writes the `Warp-SHA` trailer itself, read from the warp HEAD, before the extra trailers.
- **Rationale:** D6 rests on every weft commit carrying `Warp-SHA` (`internal/fabricengine/trailer.go`), but only `Fabric.commitWeftLocked` appends it; `CommitAnchoredPaths`, the path the drift records commit through, commits its message verbatim. The seal's `Warp-SHA` check needs the trailer on the seal commit.
- **Applies to:** cards 1, 7, 8.

### Decision: old-chairs-end-through-the-strand-table

- **Decision:** Drift-Baseline and Drift-Verify end a live chair of the removed roles `drift-baseline` and `drift-verify` through a `Deps.EndRoles` seam that stops this worktree's strands of those roles from reed's strand table, as `conflictSessionStopper` stops conflict sessions, not through `runproc.EndRole`.
- **Rationale:** D4 names `runproc.EndRole`, but no seat chair is registered in `internal/runproc` under its seat role: runproc's roles are a closed set of constants (`internal/runproc/record.go`), and a seat table's role prefix reaches no run process record, so `EndRole` with `drift-verify` would end nothing. The strand table is where the chairs' roles live; the effect D4 asks for, no pane running the old stencil, is kept. This replaces D4's named mechanism, not its effect, as the decision record's `Added after Discussion` entry "Old drift chairs end through the strand table" records.
- **Bound:** `EndRoles` stops only strands owned by this run's worktree whose role is exactly one of the roles it is told; the drift rows tell it only `drift-baseline` and `drift-verify`, which no seat takes after this change, so the new `drift-expect` and `drift-judge` chairs are never ended.
- **Applies to:** cards 16, 18, 24.

### Decision: tapestry-fix-route

- **Decision:** Under tapestry, the route that fixes code which a Stuck Drift-Verify or Drift-Recheck names is the parent's `lym loom goto --to Webster` then `lym loom resume`. Merriam, re-entered with every card done, ends its turn, its run-level `drift` entry fails on the cells, and the failure reaches the verify fixer fork or `lym webster fix-verify` through the verify-gate report (card 11). Under darn it is `lym loom goto --to Darn-Write` then `lym loom resume`.
- **Rationale:** the operator's decision 3. Webster lies outside every review segment, so `lym loom goto`'s settled-verdict guard admits it from both rows, where a goto to a settled fix row would be refused. Master edits no code, so the fix goes through the fixer paths a run-level gate failure already takes. Like darn's route, the fix is checked again by Drift-Verify and Drift-Recheck but not reviewed again.
- **Researched fact:** a re-entered Webster with every card done still starts Merriam. `loomshed`'s Webster producer has no done short-circuit (`internal/loomshed/webster.go`, `Call` hands straight to `shedadapters.NewWebsterProducer`), and `websterengine.Run` (`internal/websterengine/runlevel.go:389` to the `StartMaster` call) returns early only on errors: it archives the stale outcome and summary, removes the stale verify-gate report, writes the verify-fix prompt and starts Master with the run-level gate, whose entries run at Master's turn end.
- **How the route is derived:** Drift-Verify derives the route from `Deps.Rows`: `Darn-Write` when the run's rows hold it, `Webster` otherwise.
- **Applies to:** cards 10, 11, 16.

### Decision: no-drift-check-at-card-or-batch-gate

- **Decision:** The card and batch gate keys get no drift check. A card fork may run `lym drift check` after its card, and the guarantee stays with Merriam's run-level `drift` entry and Drift-Verify.
- **Rationale:** D8 leaves this to the plan under the operator's decision 3. The card and batch keys run only test tiers and the closed lints `comments` and `pattern`, and adding a non-test check would mean a new lint kind in `internal/gateslot` and `internal/gatecli`, run once per card, each with a full capture. The run-level entry already catches every cell before Webster ends.
- **Applies to:** cards 10, 14, 21.

### Decision: implementer-row-may-carry-gates

- **Decision:** `gatedRowFanViolations` admits a gated `Implementer` row.
- **Rationale:** D8 puts the `drift` entry on the Webster row's `gates` list, which the fork-rule test refused. Merriam already runs its run-level `verify` gate at its turn ends while it spawns card forks, so the recipe-level entry rests on the same rule that a turn end with an outstanding fork is a wait.
- **Applies to:** card 10.

### Decision: hub-only-keys

- **Decision:** `configengine.Spec` gains `HubOnlyWayForward`, and `configreg.PrepareRunSettings` refuses the keys listed there with their way forward. Drift lists `diff` and `key_patterns`, which are also classified `CategoryGuard`.
- **Rationale:** D15 refuses both keys in a task's run_settings, naming the hub's drift.yaml. The generic local-layer refusal names no way forward, and `RetiredWayForward` would call a live key retired.
- **Applies to:** card 4.

### Decision: stop-prefixes

- **Decision:** Every drift Stuck whose way forward is a parent verb (`lym drift amend`, `--restore`, `--discard`, `lym drift defer`) or a parent or operator action on the checkouts or the warp branch (the judge-change restore, the snapshot-lost check, the warp-head reset) opens with `shedadapters.DecisionHaltPrefix`, so the stop chain hands it to the parent and never to the arbiter (`PATTERN-shed-stop-chain`). One whose way forward includes `lym drift defer` carries `DeferHaltPrefix` right after it, so its reason reads `DecisionHaltPrefix`, then `DeferHaltPrefix`, then the detail; `Defer` admits a halt whose error, past an optional leading `DecisionHaltPrefix`, opens with `DeferHaltPrefix`.
- **Rationale:** `arbiterstop.HandsBack` matches only a leading `DecisionHaltPrefix` (`internal/arbiterstop/arbiterstop.go:63`), so that prefix must come first. D12 admits `defer` at named Stucks from Drift-Expect to Drift-Recheck, and the second prefix lets the verb tell those Stucks from others on the same rows, the way `CaptureHaltPrefix` does for Drift-Baseline today.
- **Applies to:** cards 8, 12, 16, 18, 19.

### Decision: counted-from-the-tracked-record

- **Decision:** Drift-Expect, `Check` and Drift-Verify judge whether the capture counted from the tracked scope record alone: `awaited_at` set without `deferred` is a counted capture, and its baseline is then checked by `CheckBaseline`, whose missing-directory fault names `lym drift defer`; no `awaited_at` is a capture that never counted. They never read `capture.json` for this.
- **Rationale:** `capture.json` lives under `.lym/drift`, so it is gone exactly when D12's lost baseline happens (`.lym` cleaned, another machine); judging from it would make the missing-baseline Stuck unreachable.
- **Applies to:** cards 10, 12, 16.

### Decision: identity-occurrence

- **Decision:** Every keyed identity carries its occurrence `N`, counted from 1, and cells match on key and `N`. A listing, a Stuck reason and the findings write `n` only for a key that occurs more than once in that file.
- **Rationale:** D1 keeps key and occurrence as separate fields. Matching on both keeps a unique key's cell stable, and writing `n` only when it disambiguates keeps the expect agent's entries short.
- **Applies to:** cards 3, 5, 10.

### Decision: amendment-entries-file

- **Decision:** `lym drift amend <slug> --entries <file>` reads the added entries from a YAML file shaped like `expected.yaml`, parsed by its sole parser.
- **Rationale:** D11 has an amendment add entries in D5's shape, validated by D5's rules. One shape and one parser cover both files.
- **Applies to:** card 19.

### Decision: pattern-line-text

- **Decision:** Card 6 fences the `PATTERN-sole-parsers` line exactly as the decision record's `Added after Discussion` entry "Short PATTERN-sole-parsers text" declares it, and the format list moves into the background file.
- **Rationale:** the originally declared text grew `PATTERN.md` 42 bytes past its headroom line; the short text names no format, so it also no longer waits on `raddle-recipe`.
- **Applies to:** card 6.

## PATTERN changes

- `PATTERN-sole-parsers` — changes: Parsing an on-disk format its background lists, or classifying a plan ref: only its sole parser.
