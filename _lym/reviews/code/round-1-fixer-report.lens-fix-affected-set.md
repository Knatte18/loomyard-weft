```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

I ran each set-premise finding's own search once; the other findings (F1, F2) are test additions with no caller or site set. The searches were F3 and F5 (callers of the helper and `PairExists` wiring) and F4 (every `createRefusal` and `ErrTaskPathTaken` site).

- **F3 and F5 (`PairWorktreeStands` and `PairExists`):** The only non-test caller is `internal/boardcli/cli.go:148`, which feeds `Config.PairExists`, and the fix covers it. The fail-open direction is now `true` there, so an unlistable repository keeps refusing a rename or project change.
- **F3 (other stat-based pair checks):** `internal/beamcli/deps.go:159`, `internal/adoptcli/deps.go:70`, `internal/battencli/overview.go:290`, `internal/battencli/beamland.go:140`, `internal/loomcli/peers.go:271` and `internal/loomcli/projectrecord.go:145` still use a plain `os.Stat` on the pair path. None of them is a rename or project-change refusal, so they are outside the finding's key.
- **F4 (`createRefusal` callers):** There is one production call, `internal/battencli/wire.go:987`, and one test call, `wire_test.go:754`. Both take the new `taskPathStands` argument.
- **F4 (other `ErrTaskPathTaken` consumers):** `internal/battencli/readoptstop.go:28` (`readoptPermanent`) only classifies the error as permanent, which stays right. `readoptWayForward` at `readoptstop.go:131` declares `taken` but, in the part I read (about 15 lines of the switch), shows no use of it. Its `ErrTaskPathTaken` branch sits further down and I did not read it, so I did not check whether its wording tells the operator to delete a registered worktree. That is outside F4's named location and the review did not ask for it.

Tool calls made (2 of 6):
1. `grep` for `PairWorktreeStands|ErrTaskPathTaken|PairExists` and the `os.Stat(...WorktreePath` sites across non-test Go files. My context held only the changed files, not the rest of the callers.
2. `sed` of `internal/battencli/readoptstop.go` lines 20-45 and 122-145, to see how the other `ErrTaskPathTaken` consumers use the error.
