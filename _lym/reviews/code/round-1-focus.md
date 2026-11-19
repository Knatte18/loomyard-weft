---
round: 1
focus:
  - "D5 merge steps (`internal/fabricengine/merge*.go`, `mergestate*.go`): does every merge verb run steps 1 to 3 under the write lock, check the already-in-progress guard before step 1, and refuse a source that changed `.lym-anchor` outside an anchor-change commit?"
  - "D4 records commit during a merge (`commit.go`, `recordsgit.go`, `MergeAbort`): does the temporary-index route move HEAD and `WarpStart` together, hold the lock across commit and reset, and treat a merge stopped before step 3 as the record specifies?"
  - "D7 replay and prime guard (`replay.go`, `pull.go`, `originreconcile.go`, the prime push guard): does replay admit only records-only commits by path, rewrite `Warp-SHA` only when it names a replayed commit, and never admit a code-changing commit on its merge trailer alone?"
  - "D6/D7 failed-push classes (`failedpush.go`, `pushclassified.go`, `statuscommit`): are `operator_unpushed` read as owed and `protected` not, does the transition seam complete only on protected-only failures, and does `landingshed` read the moved protected-branch match from fabric?"
  - "D2 clone (`clone.go`, `internal/fabriccli`): are the `--single-repo` refusals (origin holds `lym-bolt`, weft binding, second URL), the marker-push-before-`lym-bolt` order, hub-directory cleanup on refusal, and the missing-marker refusal all implemented with the stated way forward?"
  - "D3 layout: does one-repository `Add` drop the parent's run-records root from the records commit, skip junction wiring, repair and adoption, and keep `_lym`, `.lym-anchor` and weft-artifact lines out of the exclude?"
  - "D1 and `lymcwd`: does the one-repository predicate read `.lym-anchor` at the worktree root only, leave a two-repository worktree with a missing weft sibling reading as two, and keep `lymcwd`'s import set unchanged?"
  - "Scope: the diff touches `internal/gitrepo`, `internal/loomcli`, `internal/runfsm` and `internal/pairteardown` beyond the packages the record's Takes list names; does each such change serve a listed decision (D4 to D7, the statuscommit wait, the run-fsm owed class)?"
  - "Declared PATTERN changes: does the diff land `PATTERN-single-repo-hub`, the `PATTERN-run-fsm` change and the `PATTERN-cwd-resolution` overview line with their declared text, plus the background paragraphs the Constraints list?"
  - "Docs: do `docs/getting-started.md` and `README.md` carry the clone description, the `--single-repo` hazard on a two-repository hub's code repository, and the switch, reset and stash hazards in the prime?"
  - "Named constants (`lym-bolt`, merge trailer, `index.lock` retry bound, failed-push classes, `lym.primeBranch`): are they constants rather than inline literals, and is the missing or mismatched `lym.primeBranch` refusal worded as D7 states?"
---

Seeded from the decision record alone (D1 to D7, D11, D12 and the constraints).
The record's D8 to D10 are out of this part's `Takes` list, so no focus entry covers them.
