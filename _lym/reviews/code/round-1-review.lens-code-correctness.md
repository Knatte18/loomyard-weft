```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness — per-case list first, then findings.

**Per function or branch (case run, verdict)**

- `adopt.go:312` `rootGoing`, `adopt root` with a stop holding root back: `held[root]` is true, so `rootGoing` is false and no deploy runs. Correct.
- `adopt.go:312` `rootGoing`, `adopt root` when the prime is already owned here: `rootNamed` makes it true, so a behind build deploys again. This is intended behaviour, since a re-run after a failed deploy must deploy. Correct.
- `adopt.go:312` `rootGoing`, `adopt alpha` under a prime owned here: `rootGoing` is false and nothing deploys. Correct.
- `adopt.go:312` `rootGoing`, a beam adopt under an unowned prime (`owners.Prime == ""`): `rootGoing` is true, so the deploy runs. This matches the owner write, which sets the prime in that case. Correct.
- `adopt.go:331-339`, `deployStale` returns an error: `finish()` carries the mutation record and `Partial`. `result.Deploy` and `result.Slugs` are set, and the holds written earlier stay. The text "no owner was recorded and the holds stay" is accurate. `cli.go:84-87` builds `adoptFields(result)` before the error check, so the `deploy` field and log path reach the caller. Correct.
- `adopt.go:331-339`, error path: `result.Warnings` and `result.Runs` stay empty, because the reconcile warnings and `reportRuns` run after the return. The other post-hold refusals behave the same, so this is not new.
- `deploy.go:46` `deployStale`, `DeployState == nil`, `Command == ""` and `Behind == false`: returns nothing. Correct.
- `deploy.go:46` `deployStale`, `DeployState` error: returns a warning naming the hand deploy. It is appended at `adopt.go:386`. Correct.
- `deploy.go:46` `deployStale`, `RunDeploy` error: `ran` still carries the log path. The error text is complete. Correct.
- `deploy.go:67` `readDeployState`, empty `Deploy`: returns early before `Open`, the executable read and the git reads, so a hub with no deploy never errors on a missing stamp. Correct.
- `deploy.go:67` `readDeployState`, branch not in `DeployBranches`: returns an empty state. Correct.
- `deploy.go:67` `readDeployState`, detached prime HEAD: `CurrentBranch` rejects it, so the result is a warning rather than a refusal. Acceptable.
- Revision form: `vcs.revision` is the full 40-hex SHA and `CurrentSHA` is full, so `Revision == head` compares the same form. The `Label` abbreviation is used only for display. Correct.
- `deploy.go:108` `installedBehind`, empty revision: returns an error, which becomes a warning. A non-empty revision that is the head returns false. A `Modified` build of the head returns false and is not redeployed; this is a policy choice, not a bug. A `Modified` build of an ancestor returns true. The ancestor, absent-commit and off-history cases follow the table in `deploy_test.go`. Correct.
- `deploy.go:121` `runAdoptDeploy`: `MkdirAll` then `RunDeploy`, with the cwd at the prime's worktree after the reconcile moved it, so the head is current. The error, timeout and exit-code mapping is correct. It uses `context.Background()` with no interrupt cancellation; the Land row passes its own `ctx`. Acceptable.
- `deploy.go:122` `testing.Testing()` guard: matches the existing sibling pattern, for example `deps.go:225` `startSuite`. Correct.

**Findings**

1. **MEDIUM, design**, `internal/adoptcli/deps.go:228` and `internal/adoptcli/deps.go:324`.
   - The deploy replaces the lym binary by rename (the summary says so) while the adopt process keeps running.
   - On Linux `os.Executable` reads `/proc/self/exe`, and that link reads `…/lym (deleted)` once the file is renamed over.
   - Every later spawn in the same call, `startSuite` (`deps.go:228`) and the call at `deps.go:324`, then resolves a path that no longer exists, and `orchcli` does the same at `spawn.go:22` and `start.go:490`. The detached adopt check would then fail to start, as would the dobby and orch spawns.
   - The summary and `doc.go` claim "the dobby, orch and check this call starts run the new lym". Nothing in the diff pins this: no test, and no re-resolution of the path.
   - I did not confirm whether `selfexec` strips the suffix. Fix: either resolve the executable once before the deploy, or verify that the spawn helper handles the ` (deleted)` suffix, and add a regression row.
2. **LOW, design**, `adopt.go:332-338`.
   - A refused deploy returns before `reconcileWarnings` and `reportRuns`, so the envelope carries no `warnings` or `runs`. That includes any warning about branches the reconcile left unmoved.
   - This matches how the other post-hold refusals behave, and the held-run report is the less critical loss.
   - Only raise it if the handler wants the refusal envelope complete.

Tool calls:
1. `grep os.Executable/selfexec` across adoptcli, selfexec, orchcli, dobbyengine and beamcli: to see how spawns resolve the lym path after a replacement. The context held only `deploy.go`.
2. `grep` in `cli.go`: to find where `adopt()` is called and the error rendered. The context held only the diff hunks.
3. Read `deps.go:220-240`: to confirm that `startSuite` calls `os.Executable` directly.
4. Read `cli.go:82-99`: to confirm that the envelope fields are built before the error check.
