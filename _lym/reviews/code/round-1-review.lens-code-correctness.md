```yaml
lens: code-correctness
outcome: reported
```

**Lens: code-correctness. Per function and branch: the case I ran, and whether it was correct.**

- `main`
  - Empty `-hub`: fails with "-hub is required". Correct.
  - `ListProcesses` error, including `ErrUnsupported` on Windows: refuses. Correct.
  - Live process list non-empty: refuses and lists the processes. Correct.
  - `renameBolt` error after some steps ran: the message names only the failing call. Not correct, see F1.
- `liveProcesses`
  - A deleted-exe `lym` (`/home/u/go/bin/lym (deleted)`): matched after the suffix is trimmed. Correct.
  - A process with cwd inside the hub, and one naming the hub on its command line: both caught. Correct.
  - Sibling `/code/repo-LYMHUB-old`: not matched by `within`, but a command line naming it matches the `strings.Contains` check. That over-refuses, which is safe.
  - A hub reached through a symlinked path: `filepath.Abs` does not resolve symlinks, but `/proc/&lt;pid&gt;/cwd` does. `within` then misses a process working in the hub, and only the exe or command-line checks remain. Partly wrong, see F3.
  - Self and ancestors: skipped, so the `go run` parent chain does not block. Correct. An ancestor shell with its cwd inside `_board` is also skipped and is left in a vanished directory, which is minor.
- `inspectHub`
  - Pair directory `some-task` present: reported as unexpected. Correct.
  - Several prime/weft pairs: the last one wins and the other pair's directories are reported. Correct.
  - No weft, or no `_board`: reported. Correct.
  - New name present as a directory: `ReadFile` gives EISDIR, which is not `ErrNotExist`, so it is reported as "exists". Correct.
  - New name present as a file: reported as "exists". Correct.
  - `.git` or `gitdir` missing: reported through the `err != nil` case. Correct.
  - A symlink or non-`.code-workspace` file in the hub: reported as unexpected. Correct.
  - `_portals` present (it is a retired container, `internal/fabricengine/portals.go:20`): reported as unexpected, so a hub that still has one is refused. This is the stop-and-say behaviour the Design asks for, not a defect.
  - Map iteration order is random, but the messages are sorted afterwards. Correct.
- `renameBolt`
  - Normal hub: the order is checkout rename, admin rename, `.git` rewrite, `gitdir` rewrite, runproc rename, then re-point. Correct, matching the test.
  - `runproc/_board` missing: `ErrNotExist` is tolerated. Correct.
  - `.vscode` with a subdirectory: skipped by `IsRegular`. Correct.
  - File without `_board`: not rewritten and not listed. Correct.
  - `custom_board`: not touched, because a letter precedes it. Correct.
  - A later step failing: not correct, see F1.
  - `_boards`, `_board_old` or `_boardroom` in a workspace file: rewritten, because the regex has no trailing boundary. Not correct, see F2.

Findings:

**F1 — LOW / design — `tools/boltrename/main.go:57-60,177-230`**
A failure after the first `os.Rename` leaves the hub half-renamed, and the message does not say which steps completed.
- Example: the admin-dir rename fails after the checkout is already `_bolt`.
- Rerunning is refused, because `inspectHub` then reports "no _board directory".
- `main` also discards the `repointed` list on error, so the operator cannot tell which files were already rewritten.
- The Design says to stop and say what was found, with no recovery code. The fix is to name progress in the error, not to add recovery.
- Fix: wrap each step's error with its step name, for example `fmt.Errorf("after renaming the checkout, rename the admin directory: %w", err)`. In `main`, print `repointed` before failing.

**F2 — LOW / design — `tools/boltrename/main.go:32`**
`oldToken` guards only the preceding character. The regex `(^|[^A-Za-z0-9])_board` also rewrites `_boards`, `_board_old` and `_boardroom` in the workspace, `.vscode` and `reed.json` files. Only the exact directory name should be rewritten.
- Fix: add a trailing guard that does not consume it, such as `_board($|[^A-Za-z0-9_])` with `${2}` in the replacement. Alternatively, use a `(?:/|"|\s|$)` lookahead equivalent.
- Add a `_boardroom` row to the workspace fixture in `rename_test.go`.

**F3 — LOW / design — `tools/boltrename/main.go:50-52,81`**
The liveness check compares `/proc` cwds, which are symlink-resolved, against a `hubPath` from `filepath.Abs`, which is not. A hub entered through a symlinked path is therefore missed by the cwd test. The Design requires refusing when anything looks wrong, so this check should fail closed.
- Fix: after `Abs`, call `filepath.EvalSymlinks` on `hubPath` and use the resolved path for the checks. Keep the original path for the command-line match as well.

Not a finding: tracked contents of the Bolt checkout, such as a config value `path: ../_board`, are not touched by the program. I did not find such a value in non-test code (`boardcli/cli.go:91` only documents the path). I did not read the config template and could not confirm whether any exists.

Tool calls (2 of 6):
1. `grep` for `portalsDirName` in `internal/fabricengine`: to see whether hubs still carry `_portals`, which my context lacked.
2. `grep` for `BoltDirName` next to yaml, `../` or `path:` in `internal`, and for `AnchorFileName`: to see whether other files store the Bolt path, which my context lacked.
