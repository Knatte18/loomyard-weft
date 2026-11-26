```yaml
lens: bound-breaker
outcome: reported
```

Task: bound-breaker lens, attacking every bound, guarantee and claim about existing code in the round-2 decision record.

**Claims attacked**
- D13 "the encoder is shed's own writer, so the status file's bytes stay as today's": **broken** (F-a).
- D13 / D5 "the required-seam refusal reaches only a process that leaves a seam unset, such as a test binary": **broken**. The refusal reaches every package's tier1 suite through hubforge (F-b).
- D13 "with no workspace builder installed, a regeneration writes and removes no workspace file" kept within D5: **broken** for test binaries of other packages (F-c).
- Declared `PATTERN-fabric-boundary` text "only `fabricengine` runs git": **broken** by existing fabriccli code (F-d).
- D12's "`internal/fabriccli` is above fabric" against the declared PATTERN text "fabric is … fabriccli": **broken** (F-e).
- D13 tier0 production-closure guard "fails when the closure holds a package above fabric": **held**, but weaker than it reads (F-f).
- D3 "no loop runs more rounds than today's code retries; waits run through BackoffSleep from `gitrepo.PushBackoffDelays()`": **broken** for the takeover write (F-g).
- D2 "go-git refuses a reftable store at open, so it never yields a wrong ref": **held**. `git.Open` calls `verifyExtensions` (go-git v5.19.1 `repository.go:216`), and `unvalidatedBranchConfigStorage.Config` keeps `Raw` (`gogit.go:173`), so `refstorage` reaches `ErrUnknownExtension`. The `HEAD` read before it does not resolve the ref.
- D2 "no verb starts more git processes on any path than its current reads": **held**. The reftable fallback runs where today's reads already fail or fall back.
- D2 "the pending-teardown record keeps its shape": **held**. `writePendingTeardown` maps the job onto `PendingTeardown` (`pendingteardown.go:33`), whose fields are independent of the job's.
- D4 "only a failed or refused warp-branch delete is logged, as today": **held** (`add.go:860`, `add.go:866`).
- D13 "every lym process, a detached child included, runs that start": **held** for the real binary (root PersistentPreRunE with traverse hooks, `main.go:154-181`). For test binaries see F-b.
- D2 "the in-process `holdTeardown` path reads its own snapshot at the point the child would": **held** (`remoteteardown.go:339-391`).

**Findings**

- **F-a: MEDIUM, design. `_lym/discussion/decision-record.md` D13, shed writes bullet (lines 264-268).**
  - Claim: fabric reads the record's bytes, the seam's encoder returns new bytes, and fabric writes them under the lock path, so the bytes stay as today's.
  - Today: both writes are `state.WriteJSON`, which takes the status lock and then reads, merges unknown keys and writes, all under that lock (`state.go:30-41`, `state.go:77-99`).
    - The takeover writes a one-key patch type that keeps every other key (`takeoverwrite.go:58-62`, `takeoverwrite.go:228`).
    - The code-tip settle merges a `ParseStatusTolerant` status over the current file (`codetipsettle.go:90`).
  - Counterexample 1: a shed driver write to the same status file lands between fabric's read and fabric's locked write. It is lost, where today it is kept. That is a TOCTOU race today's code does not have.
  - Counterexample 2: an encoder that re-marshals a full `shedengine.Status`, as shed's writer `state.UpdateJSON` over `shedengine.Status` does (`shedrun/intents.go:22`), normalises declared keys the patch write leaves untouched. The bytes then differ from today's takeover output.
  - It also skirts `PATTERN-tolerant-records`' "rewrite via `internal/state`".
  - Fix: run the read, the encode and the write as one update under the status lock, for example a seam merge function called inside a `state`-locked update. Keep the takeover as a patch merge of fabric's own takeover type through `internal/state`.

- **F-b: BLOCKING, design. `_lym/discussion/decision-record.md` D13, required-seam bullet (line 289), and D5 (line 136).**
  - The refusal applies when the run-records paths, run locks or shed codec are unset. Every seam is installed only in `cmd/lym/main.go`'s PersistentPreRunE, inside `if !testing.Testing()` (`main.go:162-181`, confirmed by `cmd/lym/prerunseams_test.go:2`).
  - Counterexample: a tier1 test in any package builds a pair through `hubforge.AddPair`, which runs `Topology.Add` (`hubforge/hub.go:483`). `Add` adds the run-records root to its commit (`add.go:709-711`). Run paths are also used by `Remove`/`commitPendingRecords`, pull (`pull.go:587`), replay (`replay.go:276`, `replay.go:304`) and the origin reconcile (`originreconcilemoves.go:44`, `originreconcilemoves.go:548`).
  - With no seam installed, those verbs refuse, across every package whose tests use hubforge, not only fabric's.
  - The record has only fabric's own tests supply path seams (line 295). hubforge's import set is frozen by Scope Out and the arbiter entry, so it cannot install shed's adapter either.
  - Fix: decide how a test binary outside fabric gets the required seams, for example a test-support install that the record admits. Otherwise the unset meaning must keep today's behaviour.

- **F-c: MEDIUM, design. `_lym/discussion/decision-record.md` D13, workspace builder bullet (lines 292-293).**
  - Today a regeneration with no `InstallWorkspaceContent` installed still builds and writes the workspace file. `workspaceMembers` returns nil (`hubworkspace.go:123-126`), and `vscode.BuildWorkspace` still runs (`hubworkspace.go:212`).
  - Tests outside fabric that install no builder read that file:
    - `internal/beamcli/dobbyleftover_integration_test.go:119`
    - `internal/machinecli/start_test.go:197`
    - `internal/ideengine/spawn_clean_integration_test.go:193`
  - Under D13's nil meaning no file is written or removed, so these tests misread or pass vacuously. The record is silent on them.
  - Fix: state what those packages' tests install, or keep today's write with an empty member set as the nil meaning.

- **F-d: LOW, consistency. `_lym/discussion/decision-record.md` PATTERN changes, `PATTERN-fabric-boundary` bullet (line 382), and the Constraints bullet (line 318).**
  - "Only `fabricengine` runs git", stated with no read-only exemption, is false for code this task does not change:
    - fabriccli runs `gitexec.Run([]string{"branch", "--show-current"}, …)` (`internal/fabriccli/fabric.go:582`).
    - fabriccli calls `gitrepo.New(...).RemoteURL` (`fabriccli/remove.go:220`, `fabriccli/fabric.go:990`) and `CurrentBranch` (`fabriccli/anchordeps.go:550`).
    - hubforge runs gitexec/gitkit (`hubforge/hub.go`).
  - Today's background scopes the rule to mutations on warp or weft and exempts read-only verbs (`PATTERN-fabric-boundary.md:46-47`).
  - Fix: keep "on warp or weft" and the read-only exemption in the new rule text, for example "only `fabricengine` changes warp or weft through git".

- **F-e: LOW, consistency. `_lym/discussion/decision-record.md` D12 (lines 228-230) against the PATTERN changes bullet (line 382).**
  - D12 defines `internal/fabriccli` as above fabric, while the declared `PATTERN-fabric-boundary` text says "fabric is `fabricengine`, `fabriccore`, `fabriclayout` and `fabriccli`".
  - So the declared text counts a package above fabric as part of fabric, and its "all but `fabriccli` import nothing above fabric" reads two ways.
  - Fix: name one term for the bound set, for example "fabric's core packages" for the three, and keep `fabriccli` as fabric's command layer in both places.

- **F-f: LOW, design. `_lym/discussion/decision-record.md` D13 guard (line 300) and D12 definition (lines 228-229).**
  - By the definition, a package in fabricengine's production closure whose own closure holds fabricengine is an import cycle and cannot occur.
  - So the guard reduces to "the closure holds one of the four cut modules".
  - A new fabric import of a feature package that reaches none of them passes. `internal/githubclient` and `internal/slackengine` have closures free of all five (go list output above).
  - Fix: state the bound the guard gives, or extend it with an allowlist of the below-fabric packages fabric may import.

- **F-g: LOW, design. `_lym/discussion/decision-record.md` D3 (lines 103-104).**
  - Claim: the engine hands the core `gitrepo.PushBackoffDelays()` and runs each wait through `BackoffSleep`.
  - The takeover write instead uses `req.Delays` when set (`takeoverwrite.go:169-172`) and sleeps with `time.Sleep` (`takeoverwrite.go:208`), outside `SyncOptions.BackoffSleep`.
  - Counterexample: a test or caller passing `TakeOverRequest.Delays` gets a round budget other than the snapshot's, and an executor that routes waits through `BackoffSleep` changes how the takeover's waits are injected.
  - Fix: have D3 name the per-verb delay source, the takeover's `Delays` option included, and which sleep each verb uses today.

Tool calls:
1. `internal/fabriccli/fabric.go:576-592`: to confirm that fabriccli's raw `gitexec.Run` is a read of the current branch. The manifest showed only that the call exists.
