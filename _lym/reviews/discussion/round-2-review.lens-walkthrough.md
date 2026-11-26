```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: each decision run end to end on one concrete case, with the end state checked against the decision's stated purpose.

**Decisions walked**
- D1, a fixer copy written into the kept directory and then a transition's status commit: purpose met. `statusCommitPathspec` would carry the directory, since `HoldsFile` is true once `.gitignore` exists. Teardown's `commitPendingRecords` stages the configured `_lym` names.
- D2, a send-back followed by a re-run over the same file after the commit failed: purpose met. The hash in the copy's name stops a second copy, and the subdirectory staged whole carries the first one.
- D3, a Code-Review ask answered by the orch, then a takeover before the round's review is accepted, then the round's settle on the new owner: **purpose not met.** The seed brings back the record but not the reports its asks point at (F-W1).
- D4, a Finalize fixer session of two turns where turn 1 fails the gate and turn 2 passes: purpose met, with two copies and distinct sequences. The gate check's cached second evaluation could copy the same report again, depending on where the copy goes (F-W5).
- D5, case A, a fresh local landing: met. The record is written at the merge, the row before `pushRecords`, and Finalize's status commit carries both.
- D5, case B, a parent the operator squash-merged on the remote earlier, with other runs landing after it: **not met.** The landing commit names someone else's commit (F-W2).
- D5, case C, a non-beam run whose pull-request resolver now says no: **not met.** The URL is already dropped when the row is written (F-W3).
- D6, `--extract` into an existing empty directory: met on Linux, where rename(2) replaces an empty directory. **Not met on Windows**, where Go's `os.Rename` onto an existing directory fails (F-W4).
- D6, a run cancelled after a Stuck Finalize: met. `readLoomRecords` makes it a full run, and D6's Done test lists it as not landed.
- D7, the operator running `PlainArchiveTagsOnOrigin`'s printed way forward: purpose met. The tag object is re-pushed under the code-tags prefix before the plain ref is deleted, so no record is lost.

**Findings**

F-W1
- severity: MEDIUM
- class: design
- location: `_lym/discussion/decision-record.md#D3`
- summary: D3's seed restores the ask record but not its reports, so a done ask read on the new owner fails and the readers error.
  - Each done ask's `Report` holds the old anchor's absolute path, `.lym/parent-review/ask-N.report.yaml`, which the new machine lacks.
  - `SettleParentAsk`, the parent seat's gate, the PR-Review standing read and drift's parent judge all read that path.
  - Before the seed these readers found no record at all; now they find a record and error.
- evidence:
  - The report path is set to an absolute path at `internal/burlerengine/parentreview.go:390-394` (`files.ReportPath`, from `anchorPath` at :145-156).
  - It is read at `internal/burlerengine/parentreview.go:314`, `internal/burlerengine/parentseat.go:83`, `internal/landingshed/prreviewask.go:118` and `internal/driftengine/judgeparent.go:236`.
- fix: the seed also restores each tracked report copy to its live report path and rewrites each ask's `Report` to this anchor's `ReportPath(number)`. Add the read of a done ask's report after the seed to D3's deletion criterion.

F-W2
- severity: MEDIUM
- class: design
- location: `_lym/discussion/decision-record.md#D5`
- summary: on the already-landed path with no landing record, D5 writes the parent's head as the landing commit, which is not this run's commit when other work landed on the parent after the hand merge.
  - Case: the operator squash-merged on the remote, other runs landed after that, then Finalize runs.
  - The row names a later commit, its committer time and its message, contradicting D5's own "the row's landing commit names it".
  - `archiveHeldParentMerge`'s re-read has the same fault: it resets to origin's tip, which can sit past the merge that holds the work.
- evidence:
  - `internal/landingshed/finalize.go:415-419` calls `recordLanding` on every successful `Merge`, including an already-up-to-date one.
  - `internal/landingshed/finalize.go:742` reads `HeadSHA`.
  - `internal/landingshed/finalize.go:235-240` lands an already-landed parent "as for a fresh landing".
  - `internal/landingshed/finalize.go:853-854` re-reads the head after the archive.
- fix: when the merge is already up to date and no landing record exists, find the parent commit that first holds the task's work, or record the landing commit as unknown and mark the row derived. Never take the parent's head. Add a criterion: a parent the task's work reached by hand, with a later commit on top, names the hand merge or no commit.

F-W3
- severity: LOW
- class: design
- location: `_lym/discussion/decision-record.md#D5`
- summary: D5 reads the pull request URL "before any beam record is dropped", but two paths lose the URL before or after the row is written.
  - When the resolver says no, a non-beam run's own pull-request record is removed under the lock before the landing, so the row loses the URL of a pull request that was made.
  - A Finalize re-entered after Done finds the beam record already dropped, so its rewrite drops the URL and breaks the byte-identical-row claim.
- evidence:
  - `internal/landingshed/finalize.go:216` ("A no closes a recorded open pull request, drops the record").
  - `internal/landingshed/finalize_pr.go:336` and `:233-238` (`removePRRecord`).
  - `internal/landingshed/finalize.go:421-423` (`dropBeamRecord` after Done) and `:345-346` (re-entry after the done mark).
- fix: Finalize writes the URL into the landing record when it first reads the pull request, before any drop. The row reads the URL back from that record.

F-W4
- severity: MEDIUM
- class: design
- location: `_lym/discussion/decision-record.md#D6`
- summary: bound rule 4 renames the sibling onto `<dir>`, but D6 also accepts an existing empty `<dir>`.
  - Renaming a directory onto an existing directory replaces it only on POSIX; on Windows `os.Rename` fails.
  - Every extract into an existing empty directory would then take the failure path.
  - The rule rests on unpinned tool behaviour.
- evidence: D6 refuses only a `<dir>` that "exists and is not empty" (decision-record.md:108) and renames onto `<dir>` (decision-record.md:117). `internal/fsx/fsx.go:28-34` handles drive-letter paths, so Windows checkouts are in scope.
- fix: an existing empty `<dir>` is removed with `os.Remove`, which deletes only an empty directory, right before the rename. Alternatively, write into the sibling and move its entries into `<dir>`. Pin the chosen behaviour with a criterion that extracts into an existing empty directory. Also say what a leftover sibling from a killed extract does to the next extract: refuse it naming its removal, or reuse it.

F-W5
- severity: LOW
- class: consistency
- location: `_lym/discussion/decision-record.md#D4`
- summary: two places where D4's wording does not match the gate check's mechanism.
  - The gate check is evaluated again after a run has finished and answers from its cached result. A per-turn copy that allocates a new sequence would duplicate the last turn's report unless the copy runs only on a fresh evaluation.
  - D4's bound says a turn that times out never reaches the gate check, but the timeout is judged inside the check.
- evidence: `internal/landingshed/fixerrun.go:269-275` (`decided` cache and timeout halt inside `judge`) and `:229-232`.
- fix: state that the copy is made once per judged turn, after the `decided` short-circuit. Reword the bound: a timed-out turn's report is copied when the check judges it, and only a lym process killed before the check leaves a report uncopied.

**Tool calls**
1. A grep for readers of `ask.Report`, the pull-request record removal in Finalize and `AlreadyUpToDate` in fabric. My context had none of these readers or the drop site.
2. `sed` of `internal/landingshed/finalize_pr.go:225-240`, `internal/landingshed/prreviewask.go:110-125` and `internal/burlerengine/parentseat.go:78-90`, to confirm those readers fail on a missing report and to see the record removal.
