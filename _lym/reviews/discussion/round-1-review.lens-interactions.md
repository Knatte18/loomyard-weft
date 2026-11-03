```yaml
lens: interactions
outcome: reported
```

**Pairs checked (interactions lens):**

- D2 × D3 (only lym sets flag): held.
- D2 × D4 (crash between file write and flag, then remove/rename): held — D4 moves/removes the orphan file.
- D2 × D4 (prune of done entry; D2 refuses done, D4 removes its file): held.
- D2 × D5 (store commit replayed against origin that renamed/removed/changed the file): held — flag and file lost together or flag merged onto origin-held file.
- D2 × D5 (sync batching file + board.json): held — `commitDirty` takes the board write lock, so no commit between file write and save.
- D2 (done refused) × D8 (way forward = `decision store`) × D9: **broken** (F1).
- D2 bound ("only Go callers reach the method") × D9 (operator shell, arbiter strand reach it): **broken** (F2).
- D3 × D4 (merge removing slug X and upserting X again): **broken** (F3).
- D3 × D10 (get output fed back to upsert carries stored value → no-op): held.
- D4 × D5 (replayed removal/prune; set-aside of ended entries drops the file removal together with entry removal): held.
- D6 × D2 (approval on a done entry): held — unreachable state.
- D6 × D8 (decision add during Discussion-Bouncer settle): held — stated in D8 bound.
- D6 × D9 (goto past Discussion-Bouncer, stored by hand): held.
- D6 (goto skips store) × D8 (copies past approval): **broken** (F4).
- D6 store-then-Commit-fails → resume: held — resettle stores again; same/new bytes overwrite.
- D7 × D8 × D9 (one approval-row rule): held for embedded recipes.
- D10 × D2 (flag without file): held given D2 file-first.
- Auto-mode "later Discussion overwrites and keeps flag" × D8 (no copy before approval): held.
- Auto-mode older-binary orphan × D2 reader requiring flag: held.
- Constraint `PATTERN-refusal-way-forward` × D1/D2/D3/D9/D10: held, but D9's acceptance lacks the done-entry refusal it inherits from D2 (part of F1).
- Acceptance D6 × D6 paths: omits skipped-review path D6 names; not raised (no Discussion-Bouncer row carries `skip_seam` in `contracts/recipes/tapestry-recipe.yaml`).

**Findings**

F1 — LOW — consistency — decision-record.md D8/D9 vs D2
Summary: `decision add` after the entry is done fails the copy and reports `record_store_error` naming `lym loom decision store`, but D9 runs the same `StoreDecisionRecord` that D2 refuses on a done entry, so the named way forward is itself refused; neither D8 nor D9 says what a post-landing call meets, and D9's acceptance lacks the done-entry case.
Evidence: decision-record.md D2 line 43, D8 line 104, D9 lines 111–115; ledger `Ended: isDone(task)` at internal/boardengine/replay.go:252 (premise holds).

F2 — LOW — consistency — decision-record.md D2 bound vs D9
Summary: D2 bounds the run-lock bypass by "only Go callers reach the method, since no board verb writes either (D3)", but D9 adds a CLI verb reachable by the operator's shell, the parent orch and the arbiter strand, so the bypass is hand-reachable; D2's bound should name D9 as the one hand route and lean on D9's bound.
Evidence: decision-record.md line 48 vs lines 111–115.

F3 — LOW — design — decision-record.md D4 merge rule vs D3 and merge code
Summary: a merge that removes slug X and upserts X again keeps X's record file (D4 net-change), but `Store.MergeTasks` drops X from `projected` first, so the upsert builds X via `NewTask` (not `ApplyPatch`); D3 judges a carried `discussed: true` against the stored value / drops it as no-op, so the re-built entry ends with `discussed` false — a file without a flag, so `DecisionRecord` answers not found and the record is effectively lost from the entry D4 meant to keep it for. D4's acceptance asserts only that the file is kept.
Evidence: internal/boardengine/store.go:1008-1058; decision-record.md D3 line 54, D4 line 63.

F4 — LOW — consistency — decision-record.md D8 vs D6 bound
Summary: D6 says a goto past Discussion-Bouncer skips the store and names D9 as the way to store it, but after such a goto the current row is past the segment, so D8 silently copies the unreviewed record and flags the entry on the next `decision add`; D9's bound states this goto case ("a goto ... counts as past"), D8's does not, and D6's bound reads as if only D9 stores.
Evidence: decision-record.md lines 85, 101, 115.

(Outside lens, noted for completeness only: D1's `design_prefix` refusal covers only equality; a prefix such as `dec` or empty renders a design file onto a `decision-<slug>.md` name — internal/boardengine/render.go:795. Left to bound-breaker/handler.)

Tool calls:
1. grep `MergeTasks` in internal/boardengine — needed the merge semantics for remove+upsert of the same slug, not in context.
2. Read internal/boardengine/store.go:995-1064 — needed to see whether the merge re-builds the re-upserted slug via NewTask or ApplyPatch.
