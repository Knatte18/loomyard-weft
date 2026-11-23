```yaml
lens: fix-proposal
outcome: reported
```

No findings.

- **F1 (finding id per closed key):** I built the case where one ledger key maps to both a fixed and a disputed finding. The verifier names a single id, so the key closes whenever that id is fixed. This is a limit of the review's design, not a regression, and the stencil already says an entry mapping to no fixed finding stays open. An on-disk verification written without `finding` now fails the parse and the verifier re-runs, which is the fail-open path the change intends.
- **F7 (`append(judgeOutputs(...), verificationPath(...))`):** `judgeOutputs` returns a fresh slice, so the append cannot alias another slice. `rejudge` already passes the same list, including a verification that may not exist, to `archiveStaleOutputs`.
- **F8 (temporary sibling plus rename):** `.tmp` is appended to a `round-N-ledger.md` path. The two run-directory scans I found, `liveSettleRecord` and `highestCompleteRound`, only match `review.md` names through `parseRoundFileName` and `ParseRoundReviewName`, so a leftover `.md.tmp` is ignored.
- **F10 and F9:** they are text edits, and nothing contradicts them.
- **F12 (duplicate key refused):** an on-disk verification that lists a key twice falls through to a fresh verifier run, the same as any unparseable on-disk file.

Further tool calls:
1. Bash `grep` for `ReadDir`/`filepath.Glob` in `shedadapters` and `shedengine`: my context did not show whether any scan of the run directory could pick up the new `.tmp` sibling.
2. Bash `sed` on `settlerecord.go` lines 205-235 and `burler.go` lines 85-110: to confirm those two scans match only review file names, so they ignore the `.tmp` file.
