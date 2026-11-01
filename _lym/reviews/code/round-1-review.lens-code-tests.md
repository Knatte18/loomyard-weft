```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens — per behaviour of the diff, the test that pins it, or none.

**Per-behaviour list**

| Behaviour (tools/boltrename/main.go) | Pinning test | Status |
|---|---|---|
| Checkout dir renamed `_board` → `_bolt` | `TestRenameBolt_MovesTheCheckoutAndRepointsItsPaths` (gone-check on `_board`, read of `_bolt/.git`) | pinned |
| Weft worktree admin dir renamed | same test (gone-check and `gitdir` read under the new admin dir) | pinned |
| Both gitdir links rewritten (`.git` file, admin `gitdir`) | same test, `wantFiles` entries | pinned |
| `runproc/_board` record dir renamed | same test (`.lym/runproc/_bolt/12-34.json`) | pinned |
| `runproc/_board` absent is tolerated (`ErrNotExist` branch, main.go:192) | none | none |
| Re-point in `*.code-workspace`, prime `.vscode/*`, weft `reed.json` | same test; `repointed` list and contents asserted | pinned |
| `custom_board` not rewritten (`oldToken` lookbehind) | same test, workspace row `custom_board` | pinned |
| Unchanged candidate files are skipped and not listed in `repointed` | partly: `_launchers/ide-menu.sh` is not a candidate, so the skip branch (main.go:222) is never reached | none |
| Non-regular candidate (a directory in `.vscode`) skipped (main.go:214) | none | none |
| `inspectHub` stops on a stray dir (pair) | `TestInspectHub_StopsOnWhatItDoesNotExpect/PairStillInTheHub` | pinned |
| Stops when `_bolt` already exists | `.../NewNameAlreadyPresent` | pinned |
| Stops when a gitdir link points elsewhere | `.../AdminDirPointsElsewhere` | pinned |
| Stops when `_board` is missing | `.../NoBoltCheckout` | pinned |
| Stops on a stray non-dir, non-workspace file in the hub | none | none |
| Stops when there is no prime with a weft | none | none |
| Stops when the checkout's `.git` file differs (only the admin-side link is tested) | none | none |
| Stops when the new admin dir or the new `runproc/_bolt` already exists | none (`NewNameAlreadyPresent` only creates the top-level `_bolt` dir) | none |
| Accepted layout: `_launchers`, `.code-workspace` and prime/weft pass | the happy-path test | pinned |
| `liveProcesses`: lym binary (including the `(deleted)` suffix), cwd inside the hub, hub path in argv; sibling `repo-LYMHUB-old` excluded; unrelated process excluded | `TestLiveProcesses` | pinned |
| `liveProcesses`: skips own PID and ancestors | none | none |
| `main`: refuses when `proc.ListProcesses` errors, `-hub` is missing, or live processes exist | none (`main` is untested; only the `liveProcesses` filter is) | none |

**Findings**

1. **LOW, design** — `tools/boltrename/rename_test.go:93-126`: `TestInspectHub_StopsOnWhatItDoesNotExpect` leaves most of the Design's stop paths without a row. The uncovered ones are a stray file in the hub, no prime with a weft, a wrong checkout `.git` link, and a pre-existing new admin dir or `runproc/_bolt`. These are the Design's "stops and says what it found". Evidence: the `unexpected` branches at main.go:120, 138-140 and 145-150 have no matching row. Fix: add rows to the existing table; don't write new tests.

2. **LOW, design** — `tools/boltrename/rename_test.go:129-147`: `TestLiveProcesses` has no row for the own-PID or ancestor skip (main.go:76), and nothing covers the refuse-on-error or refuse-on-live path in `main` (main.go:45-51). Since "a failed check refuses" is the Design's core safety property, the `ListProcesses`-error refusal is the notable gap. Evidence: `proc.IsAncestor` is called at main.go:76 and no test row sets up that case. Fix: add a row with `PID: os.Getpid()` that is expected to be skipped. Pinning the error refusal needs `main` to be factored for it, so it can go as an accepted gap.

3. **NIT, design** — `renameBolt` has two branches no row reaches: the `runproc/_board` `ErrNotExist` tolerance (main.go:192) and the skip of files that are unchanged or not regular (main.go:214, 222). Fix: add a hub row without `runproc/_board` and a candidate file with no `_board`, as extra assertions in the happy-path test, or drop the branches.

No redundant tests found: the three test functions cover distinct behaviours. `testmain_test.go` follows the repo's `tmuxkit.Main` convention.

**Tool calls:** none made. The manifest already held the test files, `main.go` and `proc` signatures.
