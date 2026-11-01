```yaml
lens: fix-affected-set
outcome: reported
```

Findings for the lens fix-affected-set: no findings.

- **F1 (cd outside the discard):** both composers that put a directory change in a run-shell line, `composeButtonCommand` and `composeConfirmedButtonLine`, now wrap it in `DiscardOutput`. The other `ChangeDir` caller, `internal/orchcli/resumecontext.go:25`, is not a tmux run-shell line, so it needs no discard. The switch and follow lines contain no directory change.
- **Pinned strings:** no assertion elsewhere still carries the old `cd …; ` form. The only remaining pinned lines with a directory change are the updated click and confirmed rows, which include both dialects for each line.
- **F2 (pwsh click row):** the added pwsh row sits beside the POSIX one. The review's second half, an `ExitSuccess` assertion in `shell_test.go`, was already present for both dialects at `shell_test.go:346`, so adding nothing there is correct.

Calls made:
1. `grep` over `internal` and `cmd` for `ChangeDir`, `ChangeDirToWord` and `ExitSuccess` outside tests, plus `run-shell` sites: to find every line composer that uses a directory change.
2. `grep` for `lym_button_dir` and `Set-Location -LiteralPath` outside `panebin.go`: to find stale pinned strings in other tests.
