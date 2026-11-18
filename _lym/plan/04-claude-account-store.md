# Card 4 — The account file, the usage records, the pick and the resolution

**Create:**
- `plan:internal/lymdirs#ClaudeAccountsFileName` -> `const ClaudeAccountsFileName = "accounts.json"`
- `plan:internal/lymdirs#ClaudeUsageDirName` -> `const ClaudeUsageDirName = "usage"`
- `plan:internal/shuttleengine/claudeengine#accountFile` -> `type accountFile struct`
- `plan:internal/shuttleengine/claudeengine#usageWindow` -> `type usageWindow struct`
- `plan:internal/shuttleengine/claudeengine#usageRecord` -> `type usageRecord struct`
- `plan:internal/shuttleengine/claudeengine#readAccountFile` -> `func readAccountFile(dir string) (accountFile, error)`
- `plan:internal/shuttleengine/claudeengine#updateAccountFile` -> `func updateAccountFile(dir string, mutate func(accountFile) (accountFile, error)) error`
- `plan:internal/shuttleengine/claudeengine#readUsage` -> `func readUsage(dir string, accounts []AccountConfig) (map[string]usageRecord, error)`
- `plan:internal/shuttleengine/claudeengine#mergeUsage` -> `func mergeUsage(stored, read usageRecord) usageRecord`
- `plan:internal/shuttleengine/claudeengine#writeUsage` -> `func writeUsage(dir, account string, read usageRecord) error`
- `plan:internal/shuttleengine/claudeengine#recordBlocked` -> `func recordBlocked(dir, account string, until time.Time) error`
- `plan:internal/shuttleengine/claudeengine#hasHeadroom` -> `func hasHeadroom(account AccountConfig, usage usageRecord, now time.Time) bool`
- `plan:internal/shuttleengine/claudeengine#loggedIn` -> `func loggedIn(goos string, account AccountConfig) bool`
- `plan:internal/shuttleengine/claudeengine#pickRequest` -> `type pickRequest struct`
- `plan:internal/shuttleengine/claudeengine#pickAccount` -> `func pickAccount(file accountFile, req pickRequest) (accountFile, string, bool)`
- `plan:internal/shuttleengine/claudeengine#resolveAccount` -> `func resolveAccount(claude Config, worktree shuttleengine.Worktree, file accountFile, usage map[string]usageRecord, now time.Time) (AccountConfig, bool)`
- `plan:internal/shuttleengine/claudeengine#TestPickAccount` -> `func TestPickAccount(t *testing.T)`
- `plan:internal/shuttleengine/claudeengine#TestMergeUsage` -> `func TestMergeUsage(t *testing.T)`

**Extends:**
- `internal/shuttleengine/claudeengine/doc.go`

**Uses:**
- `internal/state#UpdateJSON`
- `internal/state#ReadJSON`
- `internal/shuttleengine/claudeengine#AccountConfig`
- `internal/shuttleengine/claudeengine#canonicalConfigDir`
- `internal/lymdirs/doc.go`

**Intent:**
D2, D3 and D5's stores and rules, as pure functions over the told `Worktree.AccountDir` (card 3); cards 5, 6, 9 and 13 call them.

Files (`PATTERN-lym-files`, `PATTERN-tolerant-records`): `<AccountDir>/accounts.json` and `<AccountDir>/usage/<account>.json`, named by the new `lymdirs` constants, read and rewritten only through `internal/state` with a `.lock` path beside each file, so an unknown key survives a rewrite.
A missing or empty file reads as the zero record.
So does an undecodable one, logged at Warn naming the file and that removing it is safe, since both are caches (D2); only a read error other than decoding is returned.
`accountFile` holds `Worktrees` (`worktrees`, worktree name to account name), `Picked` (`picked`, worktree name to the RFC 3339 time its entry was written, which card 5's concurrent-pick rule reads), `Counts` (`counts`, account name to a float count), `Weights` (`weights`) and `Listed` (`listed`), D3's counter.
`updateAccountFile` is one `state.UpdateJSON` hold of the file's lock.
`usageRecord` holds `Windows` (`windows`, `five_hour` and `seven_day` to a `usageWindow` of `used_percentage` and `resets_at` Unix seconds) and `BlockedUntil` (`blocked_until`, RFC 3339).

`mergeUsage` is D5's merge: per window a later reset replaces the stored one, the same reset replaces it only with a higher percentage, an earlier reset is dropped, and the stored `BlockedUntil` is always kept.
`writeUsage` merges a read into the stored record under the lock; `recordBlocked` sets `BlockedUntil` to the later of `until` and the stored value under the same lock, and keeps the windows.
`hasHeadroom` is D5's rule: false while `BlockedUntil` is in the future; otherwise true when every window the account lists has no record, a passed reset or a percentage below its threshold; windows the account does not list are ignored.

`loggedIn` is D3 step 1's test: on `darwin` every account; elsewhere a stat of `<config_dir>/.credentials.json`, reading nothing.
`pickRequest` carries the config's accounts, the usage map, the `loggedIn` predicate, the account to exclude (card 13's launched account, empty for a pick), whether step 3's fallback applies, and `now`.
`pickAccount` runs D3 over the file it is given:
- The counter reset first: an account of positive weight absent from `Listed`, or whose weight differs from `Weights`, gets the lowest count over weight among positive accounts listed then under an unchanged weight, times its own weight, or 0 with none.
- Candidates are the listed accounts of positive weight that are logged in, minus the excluded one; with none it returns not ok.
- Eligible are the candidates with headroom; it takes the lowest count over weight, ties to the earlier list entry.
- With none eligible and the fallback on, it takes the candidate whose latest blocking reset is earliest, a future `BlockedUntil` counting as one of its resets, ties to the earlier entry; with the fallback off it returns not ok.
- It writes `Counts` as reset with the taken account's count incremented, `Weights` and `Listed` from the current list, and returns the taken name; it leaves `Worktrees` to its caller.

`resolveAccount` is D4's resolution, dropping a file entry whose name the list lacks or whose account is retired (D2): the hub pin; else, for a prime or beam worktree, the beams pin unless its account has a future `BlockedUntil` and the worktree has an entry; else the worktree's entry; else none.
A hub pin is never skipped; a zero `Worktree` reaches only the hub pin.

Tests are tier0 over directories under `t.TempDir()`, each logged-in account an empty `.credentials.json` (D10):
`TestPickAccount` rows: weights 2 and 1, both eligible, six picks give four and two and never the second twice in a row; a listed window at or above its threshold with a future reset is passed over, and with its reset passed it is taken; an account listing no windows with a future `BlockedUntil` is passed over; an account without `.credentials.json` and one of weight 0 are never taken; no candidate returns not ok; every candidate short takes the earliest latest blocking reset, a tie to the earlier entry; with the fallback off it returns not ok; an added and a reweighted account get the lowest unchanged count over weight times their weight, and an account listed throughout under one weight keeps its count; the excluded account is never taken; `darwin` needs no credentials file.
`TestMergeUsage` rows: a later reset replaces, the same reset keeps the higher percentage, an earlier reset is dropped, and `BlockedUntil` survives a merge; a missing usage file reads as the zero record; an undecodable account file and an undecodable usage file read as zero records.

**ImpactSummary:** New file of unexported functions and two new `lymdirs` constants; nothing existing changes behaviour until cards 5, 6, 9 and 13 call it.

**Commit:** 4: claudeengine: account file, usage records, pick and resolution
