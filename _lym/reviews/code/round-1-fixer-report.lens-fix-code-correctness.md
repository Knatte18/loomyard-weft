```yaml
lens: fix-code-correctness
outcome: reported
```

Task: fix-code-correctness lens over the F1/F2 change set (commit c66eebf8, `tools/boltrename/main.go` and `rename_test.go`).

No findings.

Traced from the code in context:

- **`main` and `liveProcesses`:**
  - If `-hub` is a symlink `/link/h` to `/code/h`, `absPath` is `/link/h` and `hubPath` is `/code/h`. A process with cwd `/code/h/repo` now matches through `within(hubPath, p.Cwd)`. A command line naming `/link/h` matches through the new `absPath` test.
  - If `EvalSymlinks` fails (hub missing), `main` calls `fail` before listing processes, which is the correct fail-closed behaviour.
  - `inspectHub`, `renameBolt` and the output messages all use the resolved `hubPath`. `liveProcesses` is called only from `main` and the test, and both pass the new two-path signature.
- **`TestLiveProcesses`:**
  - Process 900000006 (`vim /link/repo-LYMHUB/...`) matches only through `absPath`. Dropping that clause from `liveProcesses` makes it unlisted and the test fails, so the row pins the new clause.
  - The own-PID row has exe `lym` and a cwd inside the hub, so it would be listed without the `os.Getpid()` skip. The row fails if the skip is removed.
  - The older rows are unchanged in effect. Process 900000004 (cwd `/code/repo-LYMHUB-old`) still fails `within` and has no command-line match.
- **New `TestInspectHub_StopsOnWhatItDoesNotExpect` rows:**
  - `StrayFileInTheHub`: `notes.txt` is not a `.code-workspace` file, so it lands in `unexpected` and the message names it.
  - `NoPrimeWithWeft`: with `repo` removed, no dir has a weft partner, so `h.weft == ""` and the message contains "no prime with its weft". Without that branch the message would lack the text, so the row fails.
  - `CheckoutGitLinkPointsElsewhere`: `_board/.git` reads `gitdir: /elsewhere`, which differs from the expected value. The `%q` message yields `reads "gitdir: /elsewhere"`, which matches `wantPart`.
  - `NewAdminDirAlreadyPresent`: `ReadFile` on a directory returns EISDIR, not `ErrNotExist`, so the `want == ""` branch reports `<path> exists`. The path ends in `_bolt`, so `_bolt exists` is contained. The gate run passed.

Not a finding: `main`'s `EvalSymlinks` call has no test row. `main` is untested by design, as the review's Rejected section says, and the F1 `fix` asked only for a `liveProcesses` row.

Further tool calls: none. The context already held every changed file and the gate output.
