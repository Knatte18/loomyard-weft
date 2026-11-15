```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens on the diff `055ef994..HEAD`. I ran each changed function and branch on concrete inputs.

**Per-function list (case run → verdict)**

- `runnerSessionStarter.StartSession`
  - Holder found → `HolderPID` set: correct.
  - Not found with nil error → `HolderPID` 0 and `HolderErr` nil: correct.
  - Read error → `HolderErr` set: correct.
  - Start error → returns early: correct.
- `launchOrch`
  - Empty `st.SessionID` → fresh launch with no abandoned id: correct.
  - Resume ok → `SourceResume`: correct.
  - Resume error wrapping `ErrNoTranscript` (through `resumeFailure`'s `%w` and shuttle's `%w`) → fresh launch, abandoned id set: correct.
  - Any other error → returns it, launches nothing fresh: correct.
  - Fresh launch fails after no-transcript → `report.AbandonedSession` set and error returned: correct.
- `resumeOrch`
  - `RenderReloadPrompt` error → returned: correct.
  - First start ok → returned: correct.
  - First start fails with a non-held error → wrapped by `resumeFailure`: correct.
  - Held, then `endOwnHolder` error → returned unwrapped, so `ErrNoTranscript` cannot match and nothing launches fresh: correct.
  - Held twice (the holder is the caller's ancestor, which `EndRole` spares and returns nil for) → second held error wrapped, start fails: correct, no fresh launch.
- `endOwnHolder`
  - `orchRegistered` nil → treated as unregistered and refused with the pid: correct.
  - Read error → refused with a way forward: correct.
  - Unregistered → refused naming the pid twice: correct.
  - Registered and `endOrch` error (`SurvivorsError`) → refused: correct.
  - Registered and ends 0 → logs "ended" with `ended=0`, then the second resume decides: acceptable.
- `resumeFailure`
  - nil → nil: correct.
  - Error → wrapped with `%w`: correct.
- `freshLaunchNote` with `carried=0` → "0 operator instruction(s)": correct.
- `registerOrchProcess`
  - `registerOrch` nil → "": correct.
  - `HolderPID` 0 with nil and with non-nil `HolderErr` → warning: correct.
  - Register error → warning naming the pid: correct.
- `startOrch` new lines: warnings appended before state save; registration happens before `saveFreshLaunch`, so a failed save leaves a record of a live process, which is harmless: correct.
- `wireCLI` new seams: `owner` is `HubOwner`, or the beam slug for a beam, and is used for both registration and end: correct.
- `endSubjectProcesses`
  - Prime: subject `root`, skips `HubOwner`, and beam-project owners resolve to their own slug so they are not ended: correct.
  - Beam: child entries walk up to the beam: correct.
  - Unreadable parent → logged and skipped: correct.
  - `resolvePrime`, `ReadOwners` or `Owners` error → returned: see finding 4.
  - Spare set for the `HubOwner` subject: see finding 1.
- `OwnerWatch.End` / `ownerTick`
  - nil End → skipped: correct.
  - End error → `Stop` skipped and retried next tick: see finding 4.
  - Order End → Stop: correct.
  - Race with the dobby's hand-over: see finding 2.
- `runproc.EndSubject` / `endSubject`
  - Empty runs → only the subject passes run: correct.
  - One run survives → error joined and the rest still end; `errors.As` finds the `SurvivorsError`: correct.
  - `errors.Join` of nothing → nil: correct.
  - Dobby descendants are skipped in the subject's pass and reached in the dobby pass: correct.
  - Roles ended: see findings 1 and 3.
- `RegisterOrch` / `processRecord`
  - Unreadable start time → `StartTime` "" and a path ending `&lt;pid&gt;-.json`; the lister then prunes it unless the start time stays unreadable: correct.
  - `ListStats` error → no PGID: correct.
  - `SelfRecord` still sets `Argv`: correct.
- `IsRegistered`
  - Missing owner directory → false, nil: correct.
  - Zombie holder → false, so it is refused as unregistered; misleading wording but safe: not filed.
- `checkResumable` refactor via `liveHolder`
  - dead → skip; readable start and equal → held proven; differs → skip; unreadable or empty `procStart` → held unproven. Identical to the old branches: correct.
  - The `%w: %w` wrap keeps both `ErrNoTranscript` and `os.ErrNotExist`: correct.
- `sessionHolder`
  - Missing registry → error: correct.
  - Decode or read error entry → skipped: correct.
  - Unproven live holder → returned as found: acceptable.
- `sessionRegistryDir` / prober: same path as before: correct.
- `Run.SessionHolder`
  - Engine without the capability → `0, false, nil`: correct.
  - Empty `SessionID` → none: correct.
- `Run.CarriedInstructions`: returns the run state's field: correct.
- `SessionHeldError` `Error`/`Unwrap`: delegate to `Err`; nil `Err` would panic, but no caller builds one: correct.
- `moveAnchor` spare list + `RoleOrch`: correct.

**Findings**

1. **MEDIUM, design** — `internal/runproc/end.go:103`, reached from `internal/orchcli/watch.go:153` for the prime.
   For the root subject, `EndSubject` ends every `HubOwner` record except `dobby`, `adopt` and `sync`.
   That includes `RolePush`: every non-teardown detached push registers under `HubOwner` (`internal/fabricengine/spawn.go:123-127`), so pushes started for beams still owned on this machine are killed too.
   It also ends the hub's `RoleTestMap` pass and the Slack relay.
   The doc spares adopt and sync "since they serve the whole hub", but push and test-map serve the whole hub just as much.
   `lym fabric anchor`'s prime end spares push for the same reason (`internal/fabriccli/anchor.go:468`).
2. **MEDIUM, design** — `internal/orchengine/owner.go:60-64` with `internal/orchcli/watch.go:153`.
   The watcher ends the dobby as soon as it reads the beam owned elsewhere, independently of the dobby's own probe tick.
   That can kill the dobby inside `handOver`, before `pauseRunsForOwner` and `ArchiveOwner` finish (`internal/dobbyengine/handover.go:69-80`).
   The archive of local tips origin lacks then never runs from the dobby, and nothing restarts a dobby for a beam owned elsewhere.
   Only a manual `lym beam start` retries the archive (`internal/orchcli/start.go:561`).
3. **LOW, design** — `internal/runproc/end.go:101-103`.
   `EndSubject` ends `RoleTeardown` children (`endsEveryRole` for runs, and not spared for the subject).
   Those children hold a slug's teardown lock to run a teardown's remote steps, and `pairteardown` always spares them because "only an operator's deliberate end of it stops those steps" (`internal/pairteardown/teardown.go:695-698`).
   Ending them mid-push leaves the records the new owner needs unpushed.
4. **LOW, design** — `internal/orchengine/owner.go:60-64`.
   Any persistent `End` failure now keeps the orch strand up indefinitely on a machine that no longer owns the beam, and each tick re-runs the up-to-7-second end sequence.
   Persistent failures include a `SurvivorsError` from an unkillable process, and an error from `resolvePrime`, `ReadOwners` or `Owners` at `internal/orchcli/watch.go:255-266`.
   Before the diff the strand stop did not depend on these reads.

Tool calls:
1. `internal/fabricengine/spawn.go:100-128` and `internal/battencli/arm.go:255-270`: the owner pushes and batten register under, to judge `EndSubject`'s run and hub selection. My context had only the grep lines.
2. `withdrawFirst` and `saveFreshLaunch` in `internal/orchcli/start.go`: whether the new resume-refusal path leaves the creel withdraw round inconsistent. It does not; the behaviour predates the diff and a retry re-runs the round. My context lacked these bodies.
