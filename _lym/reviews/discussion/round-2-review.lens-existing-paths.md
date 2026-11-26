```yaml
lens: existing-paths
outcome: reported
```

Task: existing-paths lens. For each state or invariant the record changes, trace the code paths that already write or read it.

**Paths traced, per decision**

- **D13, required seams** (run-records paths, run locks, shed's codec):
  - `cmd/lym/main.go:162` installs every seam only under `!testing.Testing()`.
  - `hubforge.AddPair` → `Topology.Add` → `add.go:709-710` puts `shedrun.RunsRootRel()` in the commit when `runRecordsTracked`. 121 test files outside fabricengine build hubs through hubforge.
  - **Breaks.** Every other package's test binary has no seam installed, so it meets the new "refuses before acting" path. The record gives only fabric's own tests a way to supply the seam.
- **D13, workspace builder nil** ("writes and removes no workspace file"):
  - Today with no hook installed, `RegenerateHubWorkspace` still writes the file through `vscode.BuildWorkspace` (`hubworkspace.go:212`), with members nil (`hubworkspace.go:124-125`).
  - `internal/beamcli/dobbyleftover_integration_test.go:27-33` regenerates, then reads the file, and fails on a missing one. It installs nothing.
  - `internal/ideengine/spawn_clean_integration_test.go:193-195` asserts "hub workspace file not written". It installs nothing; only `spawn_hub_integration_test.go` does.
  - `internal/machinecli/start_test.go:197` reads `HubWorkspacePath`.
  - **Breaks.** The record is silent about tests outside fabric.
- **D13, shed record writes** (bytes → encoder → write under the lock):
  - Today `codetipsettle.go:90` and `takeoverwrite.go:228` use `state.WriteJSON`. That takes the status lock, then reads the on-disk file and keeps the keys its type does not declare (`state.go:35-41`, `state.go:83-88`).
  - The takeover patch type declares only `takeover` (`takeoverwrite.go:58-62`), so today every other key of the file as it stands under the lock is kept.
  - Shed's own writer, `shedrun/intents.go:22` (`state.UpdateJSON` under `StatusLock`), writes the same file.
  - **Breaks.** D13 reads the bytes before the lock, so a shed write between that read and fabric's write is lost. The merge also leaves `internal/state`, which `PATTERN-tolerant-records` requires.
- **D13, status readers returning bytes:**
  - `pairteardown/runread.go:128` wraps `OriginRunStatus` as returning `shedengine.Status`.
  - **Survives:** the caller decodes the bytes through shed, as D13 states.
- **D13, takeover entry type:**
  - `loomcli/landingdeps.go:948` reads `ReadUpstreamTakeover`'s entry.
  - **Survives** once it is retargeted to fabric's own entry type.
- **D2, pending-teardown record:**
  - `childHold.finish` and `inProcessHold.finish` with `childEnds` both write it from the job (`remoteteardown.go`).
  - **Survives:** its shape is kept.
- **D2, in-process hold under test:**
  - `holdTeardown` takes the in-process path when `testing.Testing()`.
  - **Survives** for the snapshot rule. Whether its remote steps need a required seam meets the same gap as the first item.
- **D3, wait action through `BackoffSleep`:**
  - The takeover push loop sleeps through `time.Sleep(delays[attempt-1])`, not `BackoffSleep` (`takeoverwrite.go:208`).
  - **Survives** in production. In tests, the wait path changes.
- **D4, workspace writes through the hub `os.Root`:** **survives**.

**Findings**

1. **BLOCKING, design — D13** (`_lym/discussion/decision-record.md`, D13's required-seam bullet, plus the D5 bound).
   - Summary: the required seams are installed only by `cmd/lym/main.go`, and only outside a test binary. `hubforge.AddPair`'s `Topology.Add` uses the run-records root, and many other verbs use run paths and locks. So every package's tier1 tests that build or drive pairs through hubforge (121 test files outside fabric) hit the new "unset required seam refuses" path. The record states how only fabric's own tests supply a seam, and the arbiter entry freezes hubforge's import set.
   - Evidence: `cmd/lym/main.go:162`, `cmd/lym/prerunseams_test.go:2`, `internal/hubforge/hub.go:483`, `internal/fabricengine/add.go:709-710`.
   - Fix: decide how a test binary outside fabric gets the required seams, such as an install hook that hubforge or a test-support package calls with the real shed adapter, without breaching the frozen hubforge import set. Or make the seams' unset meaning keep today's behaviour.
2. **MEDIUM, design — D13** (workspace builder bullet).
   - Summary: with no builder installed, fabric would write no workspace file, but today a test binary gets the file from `vscode.BuildWorkspace` with nil members. Tests outside fabric that install nothing read the file and would fail: beamcli's dobby leftover test, ideengine's spawn-clean test and machinecli's start test. The record covers only fabric's own tests.
   - Evidence: `internal/fabricengine/hubworkspace.go:124-125` and `:212`, `internal/beamcli/dobbyleftover_integration_test.go:27-33`, `internal/ideengine/spawn_clean_integration_test.go:193-195`, `internal/machinecli/start_test.go:197`.
   - Fix: state what those packages' tests install, or keep a nil builder producing today's member-less file.
3. **MEDIUM, design — D13** (shed record writes sub-bullet).
   - Summary: today both writes run `state.WriteJSON`, which reads the current file under the status lock and keeps the keys its type does not declare. The takeover patch keeps every key but `takeover`. D13's sequence reads the bytes before taking the lock, then writes the encoder's output. A concurrent `shedrun` `state.UpdateJSON` write between the two is lost, and the merge moves out of `internal/state`, which `PATTERN-tolerant-records` requires.
   - Evidence: `internal/state/state.go:35-41` and `:83-88`, `internal/fabricengine/takeoverwrite.go:58-62` and `:228`, `internal/fabricengine/codetipsettle.go:90`, `internal/shedrun/intents.go:22`.
   - Fix: shape the seam as an update function that fabric runs inside `internal/state`'s locked read-modify-write. Or keep the write as a `state.WriteJSON` of a fabric-owned type with today's JSON shape, which needs no codec.

**Tool calls**

1. Counted the test files outside fabricengine that use hubforge, listed callers of the status-writing and status-reading verbs, and read the end of the beamcli dobby leftover test. My context lacked how far the seam change reaches outside fabric.
2. Read beamcli's `readHubWorkspace`, the ideengine spawn-clean assertion, machinecli's use of `HubWorkspacePath` and Add's `runRecordsTracked` condition. My context lacked how those tests read the workspace file and when Add uses the run-records root.
