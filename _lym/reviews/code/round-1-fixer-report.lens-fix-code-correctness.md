Task: check the correctness of the F1, F3, F4 and F5 code changes by tracing concrete inputs. I made no tool calls and found no BLOCKING or MEDIUM findings.

**Traces that hold**
- **`Repo.Fetch` (`pull.go`), failure paths:**
  - If the plain fetch exits non-zero, the closure returns nil, so the error is built from `code` with `failedCommand` = "fetch". The reproduction command is `git -C <path> fetch`.
  - If the second fetch exits non-zero, `failedCommand` is the full `fetch --prune --no-tags origin <specs>` line, so the reproduction command matches the failing call.
  - If `RemoteFetchRefspecs` fails with anything other than `ErrRemoteNotConfigured`, the error is wrapped and returned.
- **`Repo.Fetch`, no remote configured:** `ErrRemoteNotConfigured` returns nil, which keeps the "no remote is a no-op" row green. The gate passed in both tiers.
- **`Repo.Fetch`, default clone:** `+refs/heads/*:refs/remotes/origin/*` qualifies, so `TestFetch_BranchDeletedOnRemote_DropsItsTrackingRef` still prunes.
- **`remoteTrackingRefspecs`:** a negative refspec has no `:` and is skipped. `+refs/lym/tags/*:refs/tags/*` is skipped because its destination is not under `refs/remotes/`.
- **New fetch test:** with the old bare `--prune` in place it fails. The configured `refs/lym/tags/*:refs/tags/*` refspec would prune the local-only `set-aside/x/abc` tag, so the survival assertion would trip.
- **`TestJudgeAfterCodeTipFetch`, new row:** the first read gives `missingTip` and no error. The fetch runs, the second read returns `readFailed`, and the counts are fetch 1 and reads 2. The old row still returns before indexing `inputs`.
- **`hubRunVerdict`:** the second fetch's error is wrapped the same way as the first. A failure stays a kept wait, because `judgeAfterCodeTipFetch` swallows the fetch error.
- **`codeTipNoPairNoTip` (`act_test.go`):** it matches the `codeTipRefusal` output for empty `CodeTip` and `CodeTipMachine` with `CodeTipPairAbsent` set. The way forward ends in the bare "run the verb again once the code push lands".

**Findings**
- **LOW — `internal/gitrepo/pull.go`, the `fetchRemote = "origin"` constant.**
  - The prune pass is now tied to a remote literally named `origin`. The plain `git fetch` follows the branch's upstream remote or the default remote.
  - A checkout whose only remote has another name gets `ErrRemoteNotConfigured`, so the prune is skipped silently. Before the change, bare `--prune` pruned whatever remote was fetched.
  - Both hub store layouts use `origin` (see `initRecordsStore` in the review), so the practical impact is nil today.
- **LOW — `fetch_integration_test.go`, `TestFetch_RecordsRefspecs_PrunesTrackingRefsButKeepsLocalTags`.**
  - After `--unset-all` and the re-add, the clone's `refs/remotes/origin/main` matches the new destination pattern but has no `refs/lym/heads/main` source.
  - The prune therefore removes it, and the test never asserts on it. This is harmless and not a defect, but a reader could take that behaviour as intended.

**Calls made:** none, because my context already held every changed file.
