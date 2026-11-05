```yaml
lens: code-correctness
outcome: reported
```

Task: lens `code-correctness`, walking each changed function with one concrete input per path.

**Per-function list**

| Function or branch | Case run | Result |
|---|---|---|
| `stampRunEnd` (`rundir.go:30`) | Root exists, file absent: create, close, set mtime to `at`. | Correct. |
| `stampRunEnd` | Open fails (read-only root). | Returns the error, and `finalize` only warns. `ReadNewestRunEnd` then finds no stamp and keeps the old record-mtime fallback. Correct. |
| `stampRunEnd` | `Close` or `Chtimes` fails after the file was created. | The file's mtime is the creation time, which is about now, so the stamp is still a sane value. Correct. |
| `finalize` stamp site (`wait.go:1927`) | Every `finalize(outcome)` call (done, timeout, died, api-error, stop-marked, fork-audit failure). | The stamp is written after `saveState` and before the fork audit and cleanup, so all of them stamp. |
| `finalize` stamp site | `evaluateGate` error at `wait.go:1906`. | Returns before the outcome is saved and before the stamp. The record stays "running", so no end exists to stamp. Correct. |
| `finalize` stamp site | Stamp location versus cleanup. | `os.RemoveAll(run.runDir)` removes only the run directory. The stamp sits at its parent root, so it survives. The orphan sweep removes directories only, per the `engines-run.json` note. Correct. |
| `ReadNewestRunEnd` (`activity.go:139`) | Root present, no stamp, no runs. | Returns the zero time. Correct. |
| `ReadNewestRunEnd` | Stamp only, no records. | Returns the stamp's mtime. Correct. |
| `ReadNewestRunEnd` | Stamp older than a record, and the reverse. | `After(newest)` keeps the later of the two. Correct. |
| `ReadNewestRunEnd` | Root absent. | Returns the zero time before the stamp stat. Correct. |
| `ReadNewestRunEnd` | `os.Stat` of the stamp fails for a reason other than not-exist. | Ignored silently, so the old record-only reading remains. Acceptable, with no log. |
| `ReadNewestRunEnd` | Stale stamp from an old run, writer quiet for hours. | The stamp is old, so `judgeAgents` measures idle from it and the notice fires. It cannot hold a notice off. |
| `gateProcessLive` (`wire.go:698`) | Live `RoleGate` record. | True. Correct. |
| `gateProcessLive` | Role `loop`. | False. Correct. |
| `gateProcessLive` | Role `gate-fixer`, `verify-fixer-2`. | `IsFixerRole` matches both. Correct. |
| `gateProcessLive` | Dead pid, or a recycled pid with a different start time. | `liveness` returns `present=false`, so `List` prunes the record and it is never counted. Correct. |
| `gateProcessLive` | Zombie. | `running=false`, so it is left out. Correct. |
| `gateProcessLive` | Owner directory absent. | `List` returns nil, nil and the result is false. Correct. |
| `gateProcessLive` | Owner key. | The gate runner registers under `originSlug`, which is `location.WorktreeName` (`gatecli/run.go:408`). Fixers register under the pair slug. The reader uses `taskLocation.WorktreeName`, so another run's records are never listed. Correct. |
| `readAgentActivity` early return (`wire.go:681`) | `gateLive` true. | Returns `runs, true`. Runs are kept, so usage-limit and api-error still evaluate. `waitLive` suppresses only quiet (`runnotice.go:299,330`). Correct. |
| `readAgentActivity` (`wire.go:677`) | `runproc.List` returns an error (for example an unreadable `runproc` directory). | See finding 2. |
| `judgeAgents` / `runnotice.go:109` | One-line doc change. | Threshold logic is untouched. With no live wait and idle ≥ quiet, the notice still fires. Correct. |

**Findings**

1. **MEDIUM, design:** a fixer's pane-process record may hold the quiet notice after the fixer has finished.
   - Location: `internal/battencli/wire.go:698-709`, `internal/gatefixer/registrar.go:28-31`.
   - `Registrar` records `strand.PanePID`, which is the pane's process. `List` counts that record live for as long as the pane process runs.
   - Nothing in `internal/gatefixer` removes the strand after the reply (the grep for `RemoveStrand` and `KeepPane` found nothing). If a caller leaves the strand alive, or reed leaves a window-0 strand, `gateProcessLive` stays true and `lym` never sends a quiet notice for that run.
   - The summary claims a hung fixer delays the notice by the fixer timeout at most, and I found no bound that ends a lingering pane.
   - Unverified: whether callers always remove the fixer strand once the fixer is done.

2. **LOW, design:** an unreadable run-process directory now blocks every notice for the run.
   - Location: `internal/battencli/wire.go:677-680`.
   - `gateProcessLive` propagates the `List` error, so `readAgentActivity` returns `nil, false, err`. The `Agents` seam then fails (`runnotice.go:265`) and the whole condition returns no finding.
   - Usage-limit and api-error notices, which don't depend on this check, are suppressed with it. The new read only extends the quiet suppression, so it could log and read as not live instead.

**Tool calls**
1. `grep` for `RemoveStrand|KeepPane|Register` in `internal/gatefixer`: my context lacked whether the fixer's strand is removed after it ends.
2. `grep` for `taskWorktreeLocation` in `internal/battencli/wire.go`: my context lacked how `taskLocation.WorktreeName` is resolved, to confirm it is the slug.
