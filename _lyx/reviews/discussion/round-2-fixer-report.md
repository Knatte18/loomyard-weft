# Round 2 fixer report: discussion

Every finding was checked against the code and fixed.

## Fixed

- **F1** (MEDIUM, design): confirmed against `internal/gitrepo/gogit.go:51-76`, where today's helper holds the exclusive lock across lookup and check, and the split in D1 removes that.
  D1 now snapshots `lastPackFingerprint` under the shared lock before the read, and under the exclusive lock reruns whenever the current fingerprint differs from that snapshot, reindexing only when it also differs from `lastPackFingerprint`.
  D1's opening paragraph states the retry condition in the same terms.
  New acceptance criterion: two concurrent reads over a stale index both succeed, with one reindex.
- **F2** (LOW, design): D2's housekeeping command line now pins `gc.worktreePruneExpire=never`, so worktree admin entries stay under the destruction gate.
  D2 and D6's carve-out name the full deletion set: unreachable objects and expired reflog entries, past git's default expiry.
- **F3** (LOW, consistency): D6's D2 entry now reads "the only lyx code that runs `gc`…", and names the operator's manual `git gc` and `git maintenance start` scheduling as outside lyx's control.
- **F4** (LOW, design): confirmed `tools/deploy/main.go:107,149,360` runs `push` and `fetch` through its own `exec.Command`.
  D3's premise and Rationale now say "under `internal/`"; Scope's Out list names `tools/deploy`'s remote calls, which run only under `update-plugins.sh` in the operator's terminal.
  Support-log Interview item 3 carried the same premise and is corrected to match.
- **F5** (LOW, design): D3's Bound now states the worst case as one deadline per remote command a step attempt runs, best-effort ones included, times two attempts.
  Open risks records the residual, including 10 minutes per call on SSH.

## Disputed

None.

## Deferred

None.
