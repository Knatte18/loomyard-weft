```yaml
lens: walkthrough
outcome: reported
```

Lens: walkthrough. I ran each decision on one concrete case, passing along the data each earlier step actually produces.

**Decisions walked**

| Decision | Case | Purpose met? |
|---|---|---|
| D1 | `tapestry.implementer: weaver` set per task; the `implementers` table resolves it; the constructor gets `gates` | Held |
| D2 | Three-card plan, two batches, then verify, then Done; a NEEDS_PARENT report goes to Stuck and then the tackler | Held |
| D3 | Five cards: batch 1 is formed and run, then the rest are re-partitioned and batch 2 runs | **Broken** (finding 2) |
| D4 | Batch 2 attempt: cold strand, card gates plus row `gates`, report as output file, gate failure re-prompts | Held |
| D5 | Batch fails on sonnet, then opus[medium], then opus[high], then Stuck | Held |
| D6 | Report `FAILED` climbs; `NEEDS_PARENT` stops. The status values match the shared contract (`internal/websterengine/report.go:23-25`) | Held |
| D7 | Arbiter commits a fix, the row is re-stepped, post-batch checks run against the start commit, the batch is recorded done | Held, apart from the input-shape gap in finding 2, which `postBatchChecks` shares |
| D8 | A stop on Parent-Align reaches the arbiter; the description is built from the recipe name alone today (`internal/loomcli/arbiterprompt.go`, `arbiterRecipeDescription(recipe string)`), and D8 changes that | Held as a stated change |
| D9 | `CommitAnchoredPaths` commits `_lym/weaver` and the plan directory after each batch with a scoped pathspec; PR rework archives the run directory | **Broken** at the archive and reader end (finding 1) |
| D10 | Verify fails, a fixer climbs the ladder, verify passes, `summary.md` is written, Describe runs | **Broken** (finding 1) |
| D11 | The shared post-batch checks are given batch 2 with no recorded partition | **Broken** (finding 2) |
| D12 | Crash after the strand spawned but before state recorded it; `FindRun` attaches | Held |
| D13 | Reroute to Plan-Review after a Weaver batch: the plan gate reads the active implementer's done cards | Held for the four readers it lists; Describe is not among them (finding 1) |

**Findings**

1. **MEDIUM, design. Location: D10 and D9, with D13's reader list. Weaver's `summary.md` and its archived generations never reach Describe.**
   - D10 says Describe carries Weaver's summary "as it does Webster's", but Describe is handed Webster's paths only.
   - `RunRecordPath` is "the live generation's webster summary.md" (`internal/landingshed/describe.go:65`).
   - `PriorRunRecordPaths` come from `loomshed.ArchivedWebsterDirs`, which lists only `prior-generation/webster` (`internal/loomshed/generation.go:132-146`).
   - The archive destination is the fixed `reworkPriorWebster = "webster"` (`internal/loomshed/prrework.go:24`).
   - D9's archive of `_lym/weaver/` therefore has no prior-generation slot, and D13's list of readers to redirect leaves out Describe.
   - End state: with Weaver active, the change description loses the `## Integration suite triage` and `## Verify gate fixes` sections that D10 exists to carry.

2. **MEDIUM, design. Location: D3 against D11. The shared post-batch machinery takes the whole recorded partition, but D3 hands it one batch with no partition.**
   - In Webster, the batch number is the batch's position in `State.Partition`, recorded once at first init; "only a first init or a rebaseline replaces it" (`internal/websterengine/state.go:120-123`, `partition.go` header).
   - Report files are named by that number (`ReportFileName`, `report.go:53`).
   - The checks D11 shares read the full batch list:
     - `laterDeleteReferenceReasons(plan, batches, …)` (`recordbatch.go:190`) finds a removed target that a *later* card names;
     - `unbegunCards(batches, st)` (`recordbatch.go:114`);
     - `isLastBatch(batches, batch)` and `renderBatchGate(…, last)` (`cardgate.go:49,66`);
     - `batchCardEditError(st, bs, b batcher.Batch, …)` (`fingerprint.go:149`).
   - D3 keeps only the first batch of each partition and discards the rest. With that input, every batch looks like the last one: the last-batch gate renders for every batch, and later cards are in no batch, so the later-target check sees nothing.
   - The record gives no rule for how Weaver numbers batches or what it passes as `batches`.
   - End state: the drift and later-reference checks that D7 and D11 rely on ("caught by the post-batch drift check") do not fire for cards that are still pending.

**Tool calls**

1. `internal/websterengine/report.go` status values, to check D6's `FAILED` and `NEEDS_PARENT` against the shared report contract. The manifest had only the declaration list.
2. A grep across `internal` for `summary.md` and its readers, to follow D10's summary to Describe. Describe was not in the manifest.
3. `internal/websterengine/partition.go` and the `State` and `BatchState` fields, to check what batch shape D3 hands to D11's machinery. Not in the manifest.
