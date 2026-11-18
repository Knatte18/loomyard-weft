# Case brief: refusal-names-segment, Seed-Child producer stop

## 1. The question the next judge must answer

Why did the records push of loomyard-weft (branch `main-weft`) return without error while its commit `34b476738c933946ad51213a537c44dcfee2689d` was still unpushed, so that `recordsSideNotLanded` reported "the entry of branch main-weft is ended on origin"? Specifically: did `checkOrigin` take the `view.ended` path (push skipped, nil error) because `observeOrigin` read `main-weft` as gone from origin and `goneEntryEnded(hubPath, "main-weft")` returned true, and is that ended judgment correct for a branch that is not a run's branch? Then: is a re-run of Seed-Child safe, and does the stop need a code fix rather than a re-launch?

## 2. The facts, each with a pointer

- Stop envelope: `/home/knatte/Code/loomyard-LYMHUB/loomyard/.lym/shed/refusal-names-segment/steps/75ca90b980904100.json` is the step envelope; the drive envelope `_lym/shed/refusal-names-segment/drive-reports/drive-envelope-20261118-083839-048e.json` gives kind `producer`, first producer `Worktree-Create`, `history_length` 1, `transient` empty, current producer `Seed-Child`.
- Step trace, child trace `/home/knatte/Code/loomyard-LYMHUB/loomyard/.lym/logs/trace-20261118T073823Z-75ca90b980904100-2359104.log`:
  - line 42: `fabricengine: spawned detached push` (pid 2360314) for the `refusal-names-segment` pair, at 08:38:27.874.
  - line 46: `commit_created target=/home/knatte/Code/loomyard-LYMHUB/loomyard-weft detail=34b476738c933946ad51213a537c44dcfee2689d` at 08:38:27.897.
  - line 47: `shed: step refused`, kind producer, at 08:38:39.047, twelve seconds after the push spawned, with no failed-push record named.
- Source of the message: `/home/knatte/Code/loomyard-LYMHUB/loomyard/internal/fabricengine/pushanchored.go:309`, the fallback of `recordsSideNotLanded` (lines 297-310). It is returned from `pushRecordsSide` at line 220 when `HasUnpushed()` is still true after `pushSide` returned nil (lines 205-221). Its doc comment says "Otherwise origin ended the branch's entry", but the function never reads the ledger; the text is a generic label for "not landed".
- Route to a nil return with unpushed commits: `pushanchored.go:499-525` `checkOrigin`: `case view.gone` yields a failure, but `case view.ended: return false, nil` returns a silent skip. `awaitOrigin` (534-560) sets `view.ended` from `goneEntryEnded` when `observeOrigin` reports gone.
- `goneEntryEnded` (`pushclassified.go:633-675`): it derives `slug` from the branch name with the `-weft` suffix trimmed, so `main-weft` gives slug `main`. It takes only runs whose slug equals `main` and whose `Created` is set into `created`, then calls `readEndedSet(boardDir, upstream, []string{"main"}, created)` (line 670) and returns `len(ended) > 0`. The `readEndedSet` body was not read.
- Origin state now (read at 08:39:08): `git ls-remote origin main-weft` in `/home/knatte/Code/loomyard-LYMHUB/loomyard-weft` returns `04816744986878021040ac61ae95db94be2d9e98 refs/heads/main-weft`. Local `main-weft` and `origin/main-weft` also point at `0481674`. So origin has the branch now, and local has no unpushed commits.
- Concurrent writers to the same `main-weft` branch: the local reflog shows `c407f429da` (`batten: Open-Window -> running`, 08:38:38) and `b01637f87d` (`batten: Seed-Child -> running`, 08:38:55), both by other batten runs, landing after this step's commit.
- Stop status: `_lym/shed/refusal-names-segment/status.json`: state `failed`, `current_producer` `Seed-Child`, error the same message.

## 3. What you ruled out, and how

- Commit absent from the remote: ruled out. `git merge-base --is-ancestor 34b476738c933946ad51213a537c44dcfee2689d c407f429da5ed56e91f02406ab06b92a28a3aba4` exits 0, and `git merge-base --is-ancestor b01637f87df6e09394096caa890aed0608f4f6ea origin/main-weft` exits 0. The commit is therefore an ancestor of the current origin tip, which is the present state.
- Branch absent from the remote now: ruled out. `ls-remote` answers one line for `refs/heads/main-weft`.
- A stale read of origin's tip: the first `ls-remote` (at c407f429) and the second (at 0481674) both found the branch, so no lookup of it has failed since 08:38.
- Not checked: whether `view.gone` was true at 08:38:38-39. The child trace has no log line from `observeOrigin` or `awaitOrigin`, so the `gone` reading is inferred from the code path, not seen in the log.
- Not checked: the contents of `readEndedSet` and the `_bolt` ledger. I did not establish whether a run named `main` exists or is marked ended on origin's Bolt tip.
- Not checked: failed-push records under the hub. I did not read whether `readFailedPush` held a deferred record (the one branch that would have returned a `PushFailedError` instead).
- Not run: any `lym` verb or git command that changes state. Only read commands ran.

## 4. Hypothesis (unverified)

Hypothesis: at the push check (around 08:38:38), `git ls-remote` of `main-weft` on origin answered no line (a transient or racing lookup while concurrent batten pushes to the same branch were in flight), so `checkOrigin` took `view.gone`, `goneEntryEnded` looked up a run named `main` (which is the hub's main records branch and not a run slug), and `readEndedSet` reported it ended. The silent `view.ended` skip then left the commit unpushed, and the fallback at line 309 turned that into the misleading "entry ended on origin" error. If this holds, the defect is that `goneEntryEnded` applies run-ledger logic to a branch that is not a run branch, and that the skip at line 521-523 is silent. A plain re-launch would probably succeed now that origin has the branch, but would not remove the defect.
