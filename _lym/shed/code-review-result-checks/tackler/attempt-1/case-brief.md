# Case brief: run `code-review-result-checks`, stop `producer` at Seed-Child

## 1. The question the next judge must answer

The stop refused the records-side push of `loomyard-weft` (branch `main-weft`) with "the entry of branch main-weft is ended on origin, so the branch was not pushed". The commit that stop names is already on origin. Is this a false refusal, and if so, is a plain re-run of Seed-Child safe now that origin's `main-weft` has moved on?

## 2. The facts, each with a pointer

- Stop envelope: `/home/knatte/Code/loomyard-LYMHUB/loomyard/_lym/shed/code-review-result-checks/drive-reports/drive-envelope-20261118-095646-531a.json`: `kind: producer`, `loop.current_producer: Seed-Child`, `transient: ""`, `history_length: 1`, `state: failed`. The attempts list in the drive report is empty, so this is a first stop, not a resumed one.
- Refusal text is built at `internal/fabricengine/pushanchored.go:309` (`recordsSideNotLanded`). Its only caller is `internal/fabricengine/pushanchored.go:213-220` (`pushRecordsSide`). That path returns the refusal whenever `pushSide` returned nil and `gitrepo.New(weftPath).HasUnpushed()` is true, unless a deferred failed-push record was found. The function does not consult the Bolt ledger, so the word "ended" is not the test that fired here.
- Trace `/home/knatte/Code/loomyard-LYMHUB/loomyard/.lym/logs/trace-20261118T085625Z-80dfbc13a3f5a518-2142456.log` line 47: detached push spawned at 09:56:38.225, pid 2145419, `fabric ... --teardown-slug code-review-result-checks ... push`.
- Same trace line 51 (09:56:38.249): `commit_created target=loomyard-weft detail=d357fa4cd6d87e2b5261fa366b6f920cf077d116`.
- Same trace line 52 (09:56:46.475): `shed: step refused ... kind=producer`.
- Step trace `/home/knatte/Code/loomyard-LYMHUB/loomyard/.lym/shed/code-review-result-checks/steps/73dd0540856fd5e4.trace.log`, lines 35-51: same sequence; no in-process push result is logged between line 51 and the refusal at line 52.
- Remote read (this tackler, read-only): `git -C /home/knatte/Code/loomyard-LYMHUB/loomyard-weft ls-remote origin refs/heads/main-weft` returned `540a6c49a69e45a6650f53d980e99ab3e0c7515a`. `merge-base --is-ancestor d357fa4cd6d87e2b5261fa366b6f920cf077d116 540a6c49a69e45a6650f53d980e99ab3e0c7515a` exits 0. So the commit the stop names is present on origin. (My first `ls-remote` in the same session showed `d357fa4...` as the tip; origin moved between my reads.)
- Commits on origin `main-weft`, all by Henrik Anfinsen, Warp-SHA `0acf880f`: `d357fa4cd6` 09:56:38 (Run-Transition code-review-result-checks), `2bb7fa72e4` 09:56:39 (drift-judge-and-line-range), `540a6c49a6` 09:56:40 (scratch-symlink-two-paths).
- Local `loomyard-weft` HEAD is `1ac8836baf` (09:56:58, `batten: Open-Window -> running`, drift-judge-and-line-range). It is one commit ahead of `540a6c49`, and `540a6c49` is its ancestor. That commit is unpushed.
- `loomyard-weft` status shows `_lym/shed/code-review-result-checks/status.json` modified and `drive-reports/` and `tackler/` untracked.
- Status file `/home/knatte/Code/loomyard-LYMHUB/loomyard/_lym/shed/code-review-result-checks/status.json`: `current_producer: Seed-Child`, `state: failed`, history holds only `Worktree-Create` (done, machine `neve`).
- Bolt board on origin (`_bolt`, `origin/main`): `board.json` contains no `"main"` slug and its run for `code-review-result-checks` is at line 420.

## 3. What I ruled out, and how

- Transient or bootstrap stop: the envelope has `transient: ""` and the error is a producer error, not an environment failure.
- Commit or branch absent from remote: `ls-remote` returns `refs/heads/main-weft`, and the named commit is an ancestor of the live tip. Per the reading rule it is present, not absent.
- "Ended" by the Bolt ledger for slug `main` (from `main-weft` minus the weft suffix): `goneEntryEnded` (`pushclassified.go:633-675`) reads the ledger's ended set. No `"main"` slug appears in `_bolt` `origin/main` `board.json`, so the ledger path does not explain this refusal. Read-only ledger check, not a full replay.
- Plain re-run as a safe fix: not ruled out in the sense of being unsafe, but I cannot verify it from reading. Origin's `main-weft` has advanced with commits from other runs, and an unpushed `Open-Window` commit sits locally. A Seed-Child re-run would replay onto a moved records tip. That replay logic (`retryWeftPush`, `pushanchored.go` around line 689) needs code read and judgment.

## 4. Hypothesis (a hypothesis, not a finding)

HYPOTHESIS: the in-process `HasUnpushed()` check at `pushanchored.go:213` ran while the detached push (pid 2145419, trace line 47) had already landed `d357fa4` on origin, but the local tracking ref was still behind. Then `recordsSideNotLanded` reports a false "not landed" with no deferred record to explain it. This would make the refusal a race and a false negative, not a true "ended" entry.

Not verified. To settle it, a next judge should read: (a) the log of pid 2145419 under `/home/knatte/Code/loomyard-LYMHUB/loomyard/.lym/logs` (find the trace whose argv includes `--teardown-slug code-review-result-checks`); (b) how `pushClassified` updates the upstream tracking ref after a push (for the ordering against line 213); (c) whether the teardown or the other machine (`neve`) pushed `2bb7fa72`/`540a6c49` in the same window.
