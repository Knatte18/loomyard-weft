```yaml
lens: code-correctness
outcome: reported
```

**Task:** code-correctness lens over `9447bbda..HEAD`. For each changed function or branch, one concrete case run through the code. Result: no findings at any severity.

| Function or branch | Case run | Verdict |
|---|---|---|
| `Carrier.Tick` (`carrier.go:40`) | No request: `takeUp` takes nothing, returns nil with `running` still false. A request taken: `running` is set, then the goroutine locks `c.mu` for `setPhase` only after `Tick` has unlocked, so no deadlock. | Correct |
| `Carrier.Tick`, goroutine end | Sequence ends: `running` is cleared, then `wakeWatch` rewrites the request unchanged. The resulting event makes `Tick` call `takeUp`, which returns `errUnchanged` for a terminal request and writes nothing, so there is no event loop. | Correct |
| `takeUp` (`carrier.go:93`) | A request in `pausing` or `confirming` has its owned entry lifted and goes back to `waiting`. The reason reads the old `cur.Phase` because Go evaluates the right side of a tuple assignment first. A failed lift returns the error and writes nothing. | Correct |
| `takeUp`, re-exec | The same pid and start time match `ours`, so the request is retaken even though `daemonLive` is true. A dead other pid is taken over. | Correct |
| `HoldsReexec` and `holdReexec` (`dobbydeps.go`) | `waiting` returns false and the binary check runs. Any later phase while running returns true and the check is skipped. Covered by `TestHoldReexec`. | Correct |
| `carry` with a request taken up in `hibernating` or `resuming` | Ends at once with `failAtWake`, and the entry is lifted only if `ResumeOnWake` is set. | Correct |
| `carry` after a cancel mid-pause | `Added` records the entry, then `advance` is refused on the terminal request, so `withdrawn` lifts `kept` by identity. | Correct |
| `waitForTrigger` | A cancel makes `stillOurs` false, so it returns `(r, false, nil)`. A reading error reuses `lastReason` and doubles the delay up to 30 s. A changed reason resets the delay to 1 s. | Correct |
| `Submit` (`verbs.go`) | A pending request returns an error. A terminal request with a live kept pause is refused by `refuseKeptPause`. `DobbyLive` returning an error logs a Warn and reports the way forward. | Correct. The `found` guard is equivalent to the old `found && Terminal` because the non-terminal case returns first. |
| `Cancel` (`verbs.go`) | `hibernating` with a live carrier is refused. `hibernating` with a dead carrier is cancelled. `waiting` with a live carrier is cancelled and the sequence sees it at its next step. The re-read afterwards lifts a terminal request's owned entry. | Correct |
| `hibernateDaemonLive` (`fabriccli/anchordeps.go`) | An ended phase returns false even when the pid is alive. An empty phase or a pid of 0 falls to the pid check. A decode failure returns an error. | Correct. The ended-phase list is pinned to `TerminalPhases` by a test. |
| `dobbyWatchDirs` and `hubDobbyDeps` | A beam (non-empty slug) gets an empty request path, so no request watch and no `Tick`. The prime creates the hibernate directory before any watch opens. A failure in `NewDobbyCarrier` returns early. | Correct |
| `runStep` and `missingDirRefusal` (`steprun.go`) | An empty `Dir` resolves to the anchor. A relative `Dir` joins the worktree root. A missing directory is refused before the log and slot work, so no slot is held. The way forward is chosen by site: capture, side capture or compare. | Correct. It trusts callers to set `WorktreeDir`, and the three non-test `driftcli` and `loomcli` call sites all set it. |
| `ResolveCommandDir` and `StatCommandDir` (`gateslot/config.go`) | `""` gives the anchor, `"."` gives the root, an absolute path is kept, and a regular file in place of the directory is refused. | Correct |
| `captureReading` (`seats.go`) | The root gives `.`. A path outside the root is shown whole. A relative path error falls back to the whole path. | Correct |
| `BaselineTable` | An empty `AnchorDir` is refused by name. `loomcli` sets it. | Correct |
| `driftengine.CheckDirs` and `gateslot.CheckDirs` | An undeclared step or an empty build command is skipped. A missing dir names the key and the resolved path. | Correct |
| `checkHubModule` and `copyStagedToHub` (`configcli/hubwide.go`) | If the load fails or `CheckDirs` is nil, the load error is returned. The before/after comparison keeps the rule that a write is refused only for a new failure. | Correct |
| `newLoadFailure` (`configcli`) | Callers now pass the check result. The rollback path is unchanged. | Correct |
| `battencli` `PairDir` | It now returns the worktree path. The only consumers are the notice renderers, and `dobbynotice.go:107` already uses `WorktreePath()`. | Correct |
| `decodeStep` | An empty `Dir` stays empty. Before, `"."` joined onto the anchor gave the anchor, so behaviour is unchanged. | Correct |

Two observations, not findings:
- A sequence that ends non-terminal can only come from a failed request-file write. In that case `wakeWatch` fails too, so there is no retake loop.
- A configured drift `dir: .` now means the worktree root, not the anchor. The commit message states this and its test pins it.

Tool calls, 3 of 6:
1. `git diff` of the remaining non-test files in the range (`hibernateengine` verbs, request and wake, `configcli`, `lymdirs`, `configreg`, `battencli`, `cleanupcli`). My context had only the `hibernatecli`, `beamcli`, `fabriccli`, `drift` and `gateslot` diffs.
2. `sed` of `battencli/wire.go` around the `PairDir` seam, plus `git log -S`. I needed to know which seam changed and why.
3. `grep PairDir` over `internal`, plus `git show c896481c`. I needed the seam's consumers and the commit's stated intent.
