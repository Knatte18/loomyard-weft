```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: I attacked every bound, guarantee and claim about existing code in the decision record.

**Claims attacked**
- D1: a file in the kept directory is staged by the status commit and is not dirty outside run records. **Held**, once it joins `statusCommitPathspec` and `PathsOutsideRunRecords`.
- D1: a `.tmp-*` temp file is never staged. **Held in steady state.** It rests on unpinned git behaviour, and it breaks narrowly on first creation (B6).
- D1: teardown's commit of uncommitted records runs before the tag on every teardown path. **Held.** `remove.go:309` commits before `:328` tags. `cleanupOrphan` tags a branch with no worktree, so nothing there is uncommitted.
- D2: a re-run writes nothing new, and a copy a failed commit left behind rides the re-run's commit. **Broken on the send-back path** (B3).
- D2: the hash is computed on the send-back path. **Held**: `prfindings.go:222` computes it before the branch.
- D3: every write copies inside the same locked update. **Held.** Every writer goes through `state.UpdateJSON`.
- D3: after an adopt or takeover, every reader sees the earlier asks. **Broken for readers of the report path** (B2).
- D4: each fixer turn's report is copied, with distinct sequence numbers. **Held for collisions.** Copies can be misattributed (B7).
- D4: the fix review copy is kept whatever the outcome. **Held.** `runFixReview` has one caller, `finalize.go:1130`.
- D5: the landing commit is read as the parent's head, and an operator's hand merge is what the row names. **Broken** (B1).
- D5: a resumed Finalize leaves a byte-identical row. **Broken in a narrow window** (B9).
- D5: the row is written before `pushRecords`, so Finalize's own push carries it. **Held.** A crash before the push re-writes the row on resume.
- D6: a run that never landed is never listed as landed. **Held.** But a landed motif, pawl or sampler run is listed as not landed (B4).
- D6 bound rules 1–3 (regular files only, refuse a symlink or non-directory `<dir>`, refuse a foreign `--tag`). **Held.**
- D6 bound rule 4 (write into a temporary sibling, then rename). **Broken** by a leftover or planted sibling (B5).
- D6: never fetches and never writes a ref. **Held**: `reportengine.Open` only reads.
- D7: no code deletes an `archive/<slug>/*` tag. **Held.** The only `update-ref -d` is a remote-tracking branch ref (`gitrepo/push.go:212`). But the declared PATTERN wording conflicts with a way forward lym prints (B8).
- Constraints: the pull-request record keeps its own synchronous push. **Held**: `pattern/PATTERN-run-fsm.md:27`.

**Findings**

- **B1** — MEDIUM, design, `_lym/discussion/decision-record.md#D5`
  - On the already-landed and archive-reset paths, the record names whatever commit the local parent's head holds, which can be a later run's landing.
  - Counterexample: the operator squash-merges task T on GitHub as commit S. Run U then lands X on the same local parent. T's Finalize runs:
    - `syncParent` leaves a parent that already holds the work as it is (`finalize.go:481-483`).
    - `Merge` reports up to date.
    - No record exists, so `recordLanding` reads `HeadSHA` (`finalize.go:742`) and gets X.
  - The archive reset re-reads the head the same way (`finalize.go:853-854`), getting origin's tip, which may be past the landing too.
  - So the row's landing commit and landing time name U's commit. That contradicts D5's statement that the landing commit names "a merge the operator made by hand".

- **B2** — MEDIUM, design, `_lym/discussion/decision-record.md#D3`
  - The seed restores the ask record, but each done ask's `Report` field holds the old machine's absolute `.lym/parent-review/ask-N.report.yaml` path (`parentreview.go:146-156, 390-394`).
  - On the new owner, `SettleParentAsk` runs `os.ReadFile(ask.Report)` (`parentreview.go:314`), which fails, so settling a round accepted after the move errors.
  - D3 should also restore each report file from its tracked copy, at the path the seeded record names.

- **B3** — MEDIUM, design, `_lym/discussion/decision-record.md#D2`
  - A re-run of a send-back returns at `rec.SendBack != ""` (`prfindings.go:218-220`) before any hash or copy step. D2 doesn't order the copy against the record update (`:243-251`).
  - So a process killed between the record update and the copy leaves no copy, and the re-run D2 relies on writes none.
  - D2 should copy before the record update, or also copy on the already-sent-back path, keyed by hash.

- **B4** — MEDIUM, design, `_lym/discussion/decision-record.md#D6`
  - D6's not-landed reason and its derived line rest on reportengine's partial-run classification. That classification sorts every seed outside tapestry, darn and hardener as partial without reading its history (`runs.go:26, 265-266`), and motif, pawl and sampler are seed recipes (`shedrun/seed.go:76-81`).
  - So an older landed motif run with a Done Finalize (this task's own recipe) is listed as not landed, with the reason "the seed names the motif recipe". D6's two clauses also contradict each other for such a tag.
  - The Done test should read the status history whatever the recipe.

- **B5** — MEDIUM, design, `_lym/discussion/decision-record.md#D6` (Bound rule 4)
  - The sibling directory is named after `<dir>`, and D6 itself leaves it behind after a killed extract. The record doesn't say an existing or symlinked sibling is refused.
  - Counterexample: `<dir>.extract` (or whatever the suffix is) left over from a killed extract or planted as a symlink to `/x`. A re-run would mix stale files into the result or write through the symlink, outside the "only under `<dir>` and a temporary sibling" bound.
  - Renaming onto an existing empty `<dir>` relies on POSIX `rename(2)` replacing an empty directory. That is unpinned, and it fails on Windows.

- **B6** — LOW, design, `_lym/discussion/decision-record.md#D1`
  - The claim that the status and teardown commits skip `.tmp-*` rests on `git add -- <dir>` silently skipping ignored paths (`gitrepo/gitrepo.go:141`, `merge.go:169`, no `--force`). No test, recorded output or doc pins it, which the rubric counts as an unpinned tool fact.
  - Narrow break: the `.gitignore` is a plain, non-atomic write. Another process's state write can create a `.tmp-*` file before the ignore rule's bytes land, and a racing commit then stages it.

- **B7** — LOW, design, `_lym/discussion/decision-record.md#D4`
  - The scratch report path is cleared only at session start (`archiveStaleReport`, `fixerrun.go:129`), not per turn.
  - A turn that ends dirty (`:281-289`) or writes no new report gets the previous turn's report copied under a new sequence number, attributed to the wrong turn.
  - A re-evaluation through `s.decided` (`:269`) must not copy again.

- **B8** — LOW, consistency, `_lym/discussion/decision-record.md#PATTERN changes` / D7
  - The declared rule "archive tags are never deleted" conflicts with the way forward `PlainArchiveTagsOnOrigin` prints: `push --atomic … :<ref>`, which deletes the plain-namespace archive ref after moving it (`cleanupremotewarp.go:374-376`).
  - The rule should say tags may be moved but never dropped, or D7 should name that exception.

- **B9** — LOW, design, `_lym/discussion/decision-record.md#D5`
  - `dropBeamRecord` runs after `pushParent` answers Done (`finalize.go:421-423`). A process killed after the drop but before the Done verdict is saved re-runs Finalize with the beam record gone.
  - The rewritten row then lacks the pull request URL, which breaks the byte-identical-resume criterion.

Note for the handler: this is an incremental round, so only BLOCKING new findings on changed hunks may be raised. Each finding above sits in a hunk round 1's fixes changed.

**Tool calls**
1. Grepped fabricengine and gitrepo `git add` args, ref and tag deletes, shedrun recipe constants and PATTERN-run-fsm's pull-request push: needed to pin D1's staging claim, D7, B4's recipe set and the Constraints claim.
2. Grepped staging flags, reportengine's landing join handling of partial tags, and where teardown commits records versus tags: needed for D1's ignore claim, D6's derived line and the Constraints' "before the tag" claim.
3. Read `fabricengine/cleanup.go:262-300` (`cleanupOrphan`): needed to see whether that tagging path skips the commit of pending records.
