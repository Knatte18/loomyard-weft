```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I traced each function, branch and error path the diff adds or changes with one concrete input.

**Cases run**

- **`refuseUnseededBoard`, identity namespace or upstream present:** returns nil as before. Correct.
- **Origin read fails, for example origin unreachable:** `originBranchHead` errors. The result is the refusal plus "run the same command again once origin answers", with no push and no record. Correct.
- **Origin holds no board** (`found=false`): names the `--reset` clone. `refuseOneURLClone` refuses `--single-repo` only when `boardTip != ""`, so a records branch left on origin does not loop. Correct.
- **Origin tip equals the local tip:** `isAncestorOrEqual` returns true. The four `git config` commands are written into the board worktree, which shares its config with the records store (`clone.go:102-104`). The values match what `pushCloneSeed` writes (`destroy.go:1793-1803`). Correct.
- **Origin tip is a strict ancestor of the local tip** (local board commits made after the seed): `seedLanded=true`. Correct.
- **Origin tip absent from the local store:** `cat-file -e` returns a `GitError`, so the result is false,nil and the refusal is "another board", not an error. Correct for a foreign board. Wrong when origin moved ahead of this seed (F1).
- **Origin tip is a descendant of the local tip** (a joiner pushed after the seed landed): this reads as "another clone seeded origin". Wrong (F1).
- **Board is unborn:** `UpstreamSHA` returns `ErrNoUpstream` and `CurrentSHA` returns `ErrNoCommits`. The refusal names another board, and the message reads "this board at no commits does not contain". The classification is correct, since a seed board always has commits (`destroy.go:1737`). The wording is a NIT (F5).
- **`CurrentSHA` or `isAncestorOrEqual` fails with a non-git error:** returned bare (F3).
- **`PrimeRecordsBranchKey` missing:** `git config --get` exits 1 and the error is "read lym.primeRecordsBranch …: exit status 1", with no way forward (F3).
- **`PrimeRecordsBranchKey` set to an empty value:** exits 0 with "". The commands come out as `branch..remote` (F4).
- **`pushCloneSeed`, records push unanswered and withdrawn:** `withdrawn=true`, so the cause is wrapped. `seedPushCause` takes the default branch, and `%w` keeps the `GitError` for `seedRefusedByOrigin`. `fail` gets `recordsTip=""`, so there is no double restore. Correct.
- **Records push unanswered, origin at a third tip:** `withdrawn=false` with a nil error. The cause is unchanged and there is no claim. Correct.
- **Board push unanswered and withdrawn:** deleted, then `fail` restores records. The order and the claim are correct. The message is wrong when the restore fails (F2).
- **Board push unanswered, not withdrawn:** a board leased on absence makes `listedTip==""` match `leasedTip`, so `keep` runs. Reaching the non-withdrawn path therefore means origin holds another clone's board, and the "took" text plus "run the same clone again" mislead there (F6).
- **`withdrawUnansweredSeedPush`:** every return now carries a matching `withdrawn` value. Read error → false,keep. Withdraw error → false,keep. `pushedTip` withdrawn → true,nil. Leased tip → false,keep. Other tip → false,nil. Correct.

**Findings**

- **F1 — MEDIUM, design, `internal/fabricengine/pushclassified.go:222-232`**
  - Summary: an origin board that descends from this seed is called another clone's board. A joiner who pushed after the killed seed landed makes origin's tip a descendant of the local tip, or absent locally. `isAncestorOrEqual(originTip, localTip)` then returns false, the message falsely claims "another clone seeded origin", and it says to remove the hub. When the local board also has commits origin lacks, removing the hub discards them.
  - Evidence: `pushclassified.go:223`, and `remoteleftover.go:312-321`, where a missing commit counts as not an ancestor.
- **F2 — LOW, design, `internal/fabricengine/destroy.go:1787` with `destroy.go:1724-1729`**
  - Summary: the board-push cause states "the seed returns the records branch to what origin held before" before the restore has run. When `restoreSeededRecordsBranch` fails, for example on a lost lease, `fail` appends "restoring … also failed", so the error first claims the restore and then reports that it failed.
  - Evidence: `destroy.go:1787` and `destroy.go:1726-1727`.
- **F3 — LOW, design, `internal/fabricengine/pushclassified.go:220-221, 224-225, 233-235`**
  - Summary: three error paths drop the refusal's context and name no way forward. The `CurrentSHA` and `isAncestorOrEqual` errors are returned bare, and neither says the board was refused. A missing `PrimeRecordsBranchKey` surfaces only as git's exit status 1.
  - Evidence: `pushclassified.go:220-235`.
- **F4 — NIT, design, `internal/fabricengine/pushclassified.go:233-242`**
  - Summary: an empty `lym.primeRecordsBranch` value is not caught. `config --get` exits 0 with "", and the way forward prints `branch..remote` and `branch..merge`.
  - Evidence: `pushclassified.go:238`.
- **F5 — NIT, design, `internal/fabricengine/pushclassified.go:218-219, 229`**
  - Summary: an unborn board puts the sentinel "no commits" where a commit belongs. The message reads "which this board at no commits does not contain".
  - Evidence: `pushclassified.go:219, 229`.
- **F6 — LOW, design, `internal/fabricengine/destroy.go:1783-1789`**
  - Summary: the non-withdrawn board branch now words its cause, but this branch is reached only when origin holds another clone's board, which the `leasedTip==""` case already filters out (`destroy.go:1869`). The text "origin took the records branch, and the board's push ended without an answer" does not say so. `seedPushCause` then names "run the same clone again", which `refuseOneURLClone` refuses because origin holds a board (`recordslayout.go:220-222`).
  - Evidence: `destroy.go:1783, 1863-1872, 1920-1921`.

**Tool calls**

1. `grep` of `gitrepo` `CurrentSHA`, `UpstreamSHA`, `IsAncestor` and `ErrNoCommits`, plus `boardDir` in `clone.go`. My context lacked the unborn-HEAD behaviour of these helpers and whether the board dir shares the records store's config.
