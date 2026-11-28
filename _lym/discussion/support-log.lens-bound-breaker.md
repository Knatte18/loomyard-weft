```yaml
lens: bound-breaker
outcome: reported
```

Task: break the bounds and claims in the decision record through the bound-breaker lens.

**Findings**

1. `BLOCKING` — D5, and the D5 acceptance bullet saying the predicate is "false for tapestry". The predicate is true for tapestry, so every undiscussed tapestry start would be refused.
   - D5 defines the predicate as "its expanded rows hold a RecordWrite row with `stored_only` true".
   - D2 inlines jacquard's rows into tapestry's Jacquard block. D4 has jacquard's Record-Write row set `stored_only: true`.
   - So expanded tapestry holds that row. Counterexample: `lym batten run` of a fresh undiscussed tapestry entry, which is the normal tapestry case, would be refused by D5.
   - The predicate must also hold only when no row writes the decision record. One way is that the expansion holds no DiscussionWrite row. Another is to judge only the recipe's top-level rows, outside any block.
   - The same flaw reaches the Stuck in D4. It does not fire in tapestry only because the record already exists by then.

2. `MEDIUM` — the D6 bound: "never refuses a filing of a proposal approved before this change". This is broken.
   - `internal/loomshed/raddlefile.go:267` runs `p.deps.Check()` again at filing.
   - A proposal approved under the old binary with `recipe: tapestry` parts, and filed after the upgrade, gets `recipe-invalid` from the new vocabulary and halts the raddle run.
   - Fix either way: have the filing-time check accept tapestry, or state the halt and give its way forward.

3. `MEDIUM` — D4's way forward `lym loom goto --to Motif` is unverified. D2's in-flight claim rests on it.
   - After expansion, Motif is an engine-Block row. Its `on_done` is the sub-recipe's entry, and `SkipTo` is the exit.
   - `internal/shedengine/goto.go` records `target.Block` and has block-aware cases in its tests. Whether goto admits a block row as `--to`, and re-judges its `undiscussed` gate, is not established.
   - A goto onto Discussion-Write is the alternative. Under `PATTERN-refusal-way-forward`, the way forward needs a reaching test, so the record should name a target known to work.

4. `MEDIUM` — D2's claim that in-flight runs "resume under the new graph by row name" and that "row names stay, so on-disk identity is unchanged". This holds only in part.
   - Expansion now gives Drift-Baseline through Friction-Reflect `BlockPath` "Jacquard", where it used to be "".
   - `BlockPath` is written into `HistoryEntry.Block` (`shedengine/entry.go:55`, `reroute.go`, `goto.go`, `run.go:372`) and read by `loomcli/checklist.go`, which builds block-qualified names.
   - An in-flight run's history therefore mixes entries with no block and entries with block "Jacquard" for the same row. The record says nothing about whether checklist, report phase families or segment lookups (`checklist.go:436`) still match them.
   - Either state the effect or require a test of a resumed run's checklist.

5. `LOW` — D3's bound that "the parent's terminals must name a row that exists after expansion, which shed's validator checks". This is partly held.
   - `shedbuild/loomcontract.go` has `checkFinalizeEndsInTerminals`, so the loom contract checks the terminal chain.
   - I did not confirm a generic check that terminals exist outside the loom contract. The edge is safe for loom recipes, but the stated bound is broader than its evidence.

6. `LOW` — D7's claim that the hub reconcile carries the new blocks "as it carries any template addition" has no evidence.
   - `runprofile/doc.go:33-37` says only that a key the operator removed stays removed, and that a differing key is reported.
   - Whether a new block missing from both the base and the file is added is not shown.
   - If it is not added, raddle's `CheckProfileUnknown` refuses every jacquard part in an existing hub, so this is a fact D7 rests on.

**Claims attacked**

| Claim | Result |
|---|---|
| D2: a run parked inside Motif passes Preflight, and Record-Write writes nothing because the record and log exist | held |
| D2: a discussed entry parked at Preflight is seeded the way the Motif gate would have seeded it | held |
| D2: in-flight runs are unchanged because row names are | partly broken (finding 4) |
| D2: Preflight checks are not needed by Discussion | held; they are batcher, drift and harden, and stencils only with a Darn-Write row |
| D3: the name and segment clash checks pass, because the outside rows are only Motif and Jacquard | held |
| D3: the terminals bound | partly held (finding 5) |
| D4: an existing record is never overwritten, so added entries survive | held, per `recordWrite.Call` |
| D4: darn is unaffected | held |
| D4: the way forward is valid | unverified (finding 3) |
| D5: the predicate separates jacquard from tapestry | broken (finding 1) |
| D5: batten refuses before a worktree is made, for a run or step with no status file | held, mirroring raddle's guard |
| D5: a resume passes over the check, with Record-Write guarding behind it | held, but only for jacquard |
| D6: filing sets the discussed flag and the record | held (`boardengine/raddlefile.go:192-197`) |
| D6: an old proposal's filing is not refused | broken (finding 2) |
| D7: reconcile adds the new blocks | unverified (finding 6) |
| D8 and D9 | held; the cost-cell key change is narrow |
| D11: heddle is written only via `lym heddle` | held |

**Tool calls**

- Grepped `BlockPath` readers, terminal validation and goto's block handling. My context lacked who reads the block path and whether goto takes a block row.
- Grepped `.Block` readers in shedengine and sheddriver, the terminals checks and raddle filing's re-check. My context lacked whether history stores the block and whether filing re-runs the proposal check.
