```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I checked each pair of decisions that act on the same state, then checked each acceptance criterion and constraint against the decisions it touches.

**Pairs checked**
- D1 × D2 (kept dir, PR findings subdirectory, pr-findings commit stages the subdirectory whole, `.gitignore` honoured by `git add <dir>`): held. The pr-findings dirty check runs `git status` in the code worktree (`internal/verifytree/verify.go:274`), so an uncommitted copy in the weft's `_lym` never blocks the re-run that D2 relies on.
- D1 × D3 (ask copies written from any process under the record's lock, seed from the tracked copy, status commit stages the dir whole): held.
- D1 × D4 (per-turn and per-review copies through `internal/state`, sequence allocated from the tracked subdirectory, so numbering survives a machine move): held.
- D1 × D5 (landing record and index row in the kept dir, staged by `pushRecords`' CommitStatus): held.
- D1 × D6 (extract writes the kept dir, its `.gitignore` included): held.
- D1/D2/D3/D4 × teardown's `commitPendingRecords` (`pathspecNames` always includes `_lym`, `internal/fabricengine/junctionnames.go:26`): held.
- D2 × D3 (PR-Review findings and the PR-Review ask both kept, in separate subdirectories): held.
- D3 × D5 (index row lists the kept-record paths; an ask answered after the row is written): broken, see finding 2.
- D4 × D5 (fixer and fix review copies made before the landing, including a fixer run on the merge-in-required retry pass, which frees the lock and re-runs the pass): held.
- D5 landing record × `archiveHeldParentMerge` re-record and resumed already-up-to-date merge: held.
- D5 × D6 (row present or absent; derived line only with Finalize Done; ordering by row then tip time): held. D6's derived line also covers a row whose write failed.
- D5 × Constraints ("new records reach origin only through … the background pusher, except D2"): broken, see finding 1.
- D6 × D7 (tags listed, never deleted): held.
- D7 × PATTERN changes (`PATTERN-pair-remote` "archive tags are never deleted") × the plain-tag way forward lym prints: broken, see finding 3.
- Acceptance criteria D1–D7 against their decisions: consistent. The "D2, D3, D4 kept file after the last records commit is in the tag" criterion holds through `commitPendingRecords` staging `_lym`.

**Findings**

1. LOW, consistency, `_lym/discussion/decision-record.md#Constraints`
   - Summary: the Constraints say new records reach origin only through the run's records commits and the background pusher, with D2 the one exception. D5 sends its index row and landing record through Finalize's own records push, which is synchronous: `Finalize.pushRecords` calls `Deps.PushRecords` and waits, and an owed push comes back as a transient error.
   - Evidence: `internal/landingshed/finalize.go:712-731`; `internal/loomcli/landingdeps.go:961-979` (`runRecordsPusher` calls `pairteardown.PushRun` and waits).
   - Fix: name Finalize's synchronous records push in the Constraints exception beside D2's.

2. LOW, consistency, `_lym/discussion/decision-record.md#D5` (stale rejection in `support-log.md` Turn 3)
   - Summary: D5 now writes the index row before `pushRecords`, not as Finalize's last step. An unreached ask can still be answered "at any time" after the landing, and D3 then writes a report copy into the asks subdirectory after the row was written. The row's list of kept-record paths misses that copy, and the support log's rejection rests on "D5 now runs last", which is no longer true.
   - Evidence: `internal/burlerengine/parentreview.go:363-364` and `:371-372` (an unreached latest ask is answered at any time); `decision-record.md` D5 ordering (lines 85, 88); `support-log.md:29`.
   - Fix: state in D5 that the row's path list is a snapshot at landing and that D6 reads the kept dir itself, or list the asks subdirectory rather than its files.

3. LOW, consistency, `_lym/discussion/decision-record.md#D7` and `#PATTERN changes`
   - Summary: the declared `PATTERN-pair-remote` text says "archive tags are never deleted", and D7 says the plain-namespace listing "deletes nothing". But that way forward, which lym itself prints, ends in `push --atomic … :<plain ref>`, which deletes the plain `refs/tags/archive/...` ref on origin after copying it under the code-tags prefix. So lym's own guidance tells the operator to delete an archive tag ref, against the unconditional rule.
   - Evidence: `internal/fabricengine/cleanupremotewarp.go:374-376`.
   - Fix: bound the rule in D7 and its PATTERN text to the tag's target and contents, for example "never deleted, only moved within origin's namespaces", or state the plain-ref move as the one sanctioned exception.

**Tool calls**
- Grep of `runRecordsPusher`, `verifytree.DirtyPaths` and `structuralCommittedDirs`: the manifest held none of them. I needed them to judge whether Finalize's records push is synchronous, whether kept-dir copies show as dirty in pr-findings, and whether teardown's pending-records commit stages `_lym`.
