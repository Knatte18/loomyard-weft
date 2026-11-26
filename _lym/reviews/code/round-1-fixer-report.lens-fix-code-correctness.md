```yaml
lens: fix-code-correctness
outcome: reported
```

**Task:** fix-code-correctness check of the change set in `tools/machinecost/` (commits 6e60b0c56 and c3bd1d6bb).

I found no wrong result, broken caller or added test that would pass with its defect in place. I traced these by reading the code and did not run them with the defects put back. Two edge cases:

1. **LOW** — `main.go` package doc: "It deletes every one of them before it exits, also when an interrupt or SIGTERM ends the run early. A SIGKILL leaves the folders inside the -dir folders behind."
   - This reads as if SIGKILL were the only way to leave folders behind.
   - `signal.NotifyContext` catches only `os.Interrupt` and `syscall.SIGTERM`. SIGHUP (a closed terminal or dropped ssh session, plausible during a long probe) still ends the process without the defers or `lymtemp.Remove`, leaving the `machinecost-*` folders in the `-dir` folders.
   - Either add `syscall.SIGHUP` to the `NotifyContext` call or name it in that sentence.

2. **LOW** — `newbinary.go` `linkProbeProgram`, `exec.CommandContext(ctx, goBinary, "build", ...)` with `GOTMPDIR=`+tempDir.
   - On a SIGTERM sent to `machinecost` alone, cancellation SIGKILLs only the `go` process.
   - A running `compile` or `link` grandchild writes to a pipe of `go`'s, not ours, so `CombinedOutput` need not wait for it. It can still be writing under `tempDir/go-build*` while `os.RemoveAll(tempDir)` runs.
   - The worst case is a failed `RemoveAll` of the temp folder, which sits under the lym temp root. The `-dir` folders are unlikely to be affected, and Ctrl-C signals the whole process group, so it does not hit this. I see no change worth making for it.

Tool calls made: none.
