```yaml
lens: existing-paths
outcome: reported
```

Task: the existing-paths lens over the Weaver decision record.

**Paths traced, per decision**
- D1 guard reads Webster's state through `websterengine.LoadState`. A missing file returns nil, nil, so it counts as no batches; a corrupt file returns an error, so it refuses. Survives.
- D3's recorded full partition against `ExecutionBatches` and `mapPartition` (every plan card must sit in a recorded batch). Survives, because D3 records the done batches plus the fresh partition.
- D3/D11 batch numbering against `batchIdentity`, `unbegunCards` (`st.Batches[number]`) and `reset.go:313`. **Breaks** (finding 3).
- D3/D11 `isLastBatch` on the recorded partition. Survives: only the begin whose fresh partition holds a single batch reads as last.
- D2 `lym weaver reset` against D9's per-batch commit through the records-commit path (`fabricengine.CommitRecordsPaths`, which commits in the weft worktree). **Breaks** (finding 1).
- D13 merge-in warning through `websterengine.RunInFlight`. **Breaks** (finding 2).
- D13 Describe prior readers (`ArchivedWebsterDirs` reads only `prior-generation/webster`). The record extends it to every implementer slot. Survives as stated.
- D13 PR rework archive (`ArchiveWebster` closure in `loomcli/wiring.go:716` calls `ArchiveRunRecord(websterGeom, …)`). The record moves it to the active implementer's directory. Survives as stated.
- D13 decision notice (`decisionNoticeInline`, `heldBackReason` keyed on `MerriamStrandRole`). The record adds Weaver's roles to both. Survives.
- D13 plan gate (`loomshed.ValidatePlan` → `DoneCards`) and renumber (`LastBegunCard` under `AcquireStateMutation`). Both read `State.Batches`, which D11's shared state keeps. Survives.
- D12 attach against shuttle `Start`'s opportunistic orphan sweep and `AttachGated`'s superseded-strand removal. A swept or removed run reads as gone, and D6 then treats it as a machinery loss. Survives.
- D6 failed-attempt stop through `Runner.StopStrand`/`stopRecorded`. A record whose output files all exist keeps `running`. Survives, since D12 records finished reports first.

**Findings**

1. **MEDIUM, design. D2 and D9: `lym weaver reset` meets D9's per-batch records commit.**
   - D9 commits `_lym/weaver/` and the plan directory "through the same records-commit path Webster's run commit uses". That path, `fabricengine.CommitRecordsPaths`, commits in the weft worktree, not on the task code branch.
   - D2's reset moves "the task branch back to the run's start commit" and claims "the plan returns with the branch to its state at the run's start". A code-branch move leaves the weft commits of `_lym/plan/` and `_lym/weaver/` in place, so that claim does not hold. The record never says which tree D2's move, D6's "tree unclean" check and D9's "puts it on the branch" refer to.
   - If the reset instead moves a branch that holds the per-batch commits, the move removes the committed `_lym/weaver/` before D2's "then archives `_lym/weaver/`", so the archive finds nothing and the journal is lost.
   - Evidence: `internal/fabricengine/commitweftpaths.go:21-41`; decision record D2 lines 45-46, D9 line 121.

2. **MEDIUM, design. D13 merge-in warning: Weaver has no done marker for `RunInFlight` to read.**
   - `websterengine.RunInFlight` counts a run in flight when `state.json` exists and `outcome.yaml` is absent or not done.
   - D9's run directory has a state file, reports, `journal.md` and `summary.md`, and no outcome file. Weaver's state file stays committed after Done.
   - The record says merge-in "reads the active implementer's state" but never says what marks a Weaver run finished. Ported as-is, the rule would warn on every finished Weaver run; left unported, nothing defines in-flight.
   - Evidence: `internal/websterengine/inflight.go:11-43`; decision record D9 line 117, D13 line 176.

3. **MEDIUM, consistency. D3 and D11: D3's batch numbering does not match the shared checks' batch key.**
   - D3 numbers the fresh partition's batches "after the done ones", which reads as an ordinal count.
   - The shared machinery keys a batch record by its first card's number. `batchIdentity` returns `b.Cards[0].Number`; `unbegunCards` looks up `st.Batches[number]` with that key; `reset.go` matches `cardNumberInt(pb.Cards[0]) == batch`.
   - If Weaver keys `Batches` by an ordinal, `unbegunCards` treats begun batches as unbegun, or begun ones as pending, and the later-delete check D3 relies on reads the wrong set.
   - The record should say the batch key is the first card's number, as the shared checks require.
   - Evidence: `internal/websterengine/runlevel.go:982-987`, `recordbatch.go:114-124`, `reset.go:313`; decision record D3 line 54.

**Tool calls**
1. grep in `websterengine/reset.go`, for the order of archive and branch move, which the manifest did not cover.
2. grep for `RunRecordPath`, the records-commit helper and the arbiter description source, none of which the manifest held.
3. grep for `ArchiveRunRecord` callers and the Describe readers, for the archive and reset paths.
4. grep for `ArchiveRunRecord(` call sites, to find who archives Webster's directory.
5. Read `loomcli/wiring.go:690-734`, for the PR rework archive wiring.
6. Read `fabricengine/commitweftpaths.go:1-45`, to find which tree the records commit writes to.
