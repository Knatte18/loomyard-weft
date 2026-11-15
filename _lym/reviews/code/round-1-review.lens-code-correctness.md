```yaml
lens: code-correctness
outcome: reported
```

Task: review the correctness of the changed code in this diff (lens code-correctness).

**Per-function list (case ran, verdict)**

- `heldByPause`, no env mark, empty strand, zero `DetachedChild`: returns false, so `ExemptAll` is true and the verb passes. Correct.
- `heldByPause`, `child != DetachedChild{}`: held. This works because `DetachedChild` is a flat, comparable struct. Correct.
- `heldByPause`, strand `ly:orch`: `MatchesRole(orch)` is true, so it returns false and the orch passes. Correct.
- `heldByPause`, strand `ly:task:writer`: held. Correct.
- `heldByPause`, unparsable strand `not a strand`: `err != nil`, so held. Correct but deliberate. Any junk `LYM_STRAND_NAME` leaves an operator verb held. Fail-closed is defensible.
- `heldByPause`, env mark set and strand `ly:orch`: the mark wins and the process is held. An orch's own verb is held if the orch process inherited the mark. `stripIdentityEnv` and the `tmuxkit` filter are meant to prevent that. Correct only if every pane-spawn path scrubs.
- `InstallNetworkGate`, `runCommand` true: `os.Setenv` runs before the hub check, so it marks even outside a hub. The mark is only a child-inheritance marker, so that is harmless. In-process tests carry the mark across tests, but the tests use `t.Setenv`. Correct.
- `InstallNetworkGate`, `runCommand` true: `heldByPause` reads `os.Getenv(RunProcessEnv)` after the `Setenv`, so the run command itself is held. Correct.
- `InstallNetworkGate`, `runCommand` false, nested spawn: an operator verb (`lym s` → `beam start`) starting a run loop. The loop is `lym shed run` or `beam dobby`, which are annotated, so it marks itself. Correct. A run started in a path with no annotated command, for example a `batten`/`loom` verb that runs the loop in-process, gets no mark. The test table lists `batten run`, `batten step` and `loom run` as true, and they appear to be built on `shedverbs` (`run.go` and `step.go` carry the annotation). I did not verify `loom run` independently.
- `ExemptAll: !heldByPause`: the `ExemptAll` doc says "every live entry passes". The waiter honoring `ExemptAll` for in-flight marking is stated in `doc.go`. Not re-read.
- `runCommand(cmd)`: it reads only `cmd.Annotations` of the executing leaf. A nil map reads "" and gives false. Correct.
- `stripIdentityEnv`, env `LYM_RUN_PROCESS=1`: dropped. `LYM_PARENTAL` is kept (key compare, not prefix). Correct.
- `tmuxkit.ServerEnviron`, `LYM_RUN_PROCESS`: dropped. It is a string literal, so it can drift from `pauserecord.RunProcessEnv`; a test pins the engines against the kit but not the literal against the constant. The test sets `"LYM_RUN_PROCESS=1"` literally in both, so a rename of the constant would not fail either one. LOW.
- Pane agents: a pane's agents have no mark (scrubbed). An agent is held by its non-orch `LYM_STRAND_NAME`. Correct. An operator verb run by a non-orch agent, for example `selfreport` or `board get` from a writer strand, is held. This is the entry's "agent worktree" case, and `SetGateExemptAll` callers still cover adopt, teardown and batten cancel. Per the entry ("a strand's agents" are meant to be held) this is by design.
- `battenshed.holdSpawnWhileCovered` / `coverage`: `deps.Pause` is `pauserecord.ReadCoverage(geometry.RecordPath, slug, ...)` in `battencli/wire.go:1278`. It reads the record directly and does not consult the gate's `ExemptAll`, nor the process kind. A run-scoped pause covering the slug therefore still holds the spawn in any process that runs `InnerRun`. That process is the `shed run`/`step` of the run, not an operator verb, so `lym s`'s `beam start` itself does not reach it (it spawns the dobby, which is annotated). The entry's `lym s` complaint is whether `beam start` or `reed attach` held. Neither calls `InnerRun` directly. I could not confirm the original held line's source. Not a correctness defect in the diff, but the entry's second hold (`innerrun_pause.go`) was left unchanged. I'm flagging it as an uncertainty for the handler and filing no finding.
- `battencli/drive.go:269` `PauseHold`: `WaitUncovered` is called directly with `Waiter{}`, with no gate involved. This is the run's loop, so holding is intended.

**Findings**

1. LOW, design. `internal/testkit/tmuxkit/tmuxkit.go:540` holds the literal `"LYM_RUN_PROCESS"`, not `pauserecord.RunProcessEnv`. The pinning test (`shuttleengines_test.go:223,230`) uses the literal too, so a rename of the constant drifts silently and a run mark would leak into panes. The package comment says tmuxkit cannot import the engines. The commit message says it was kept off `pauserecord` to avoid an import cycle. The fix is to have the pinning test (external to the kit) build its environ and want-list from `pauserecord.RunProcessEnv` (`shuttleengines_test.go` already imports `pauserecord`'s neighbours), so the constant and the kit's literal are pinned together.

2. LOW, design. `heldByPause` (`internal/tendercli/gate.go:121`): `strand != ""` with an unparsable name reads as held. An operator whose shell has a stale or garbage `LYM_STRAND_NAME` has every verb held by a pause. This is fail-closed and tested (`UnparsableStrandHeld`), so I'm reporting it as an observation, not a defect.

No BLOCKING or MEDIUM defects found in the changed lines.

**Tool calls**
1. `sed` of `internal/battencli/wire.go:1270-1295` and `drive.go:262-285`, to confirm that `innerrun_pause`'s `deps.Pause` reads the record directly and does not consult the gate's `ExemptAll`; the manifest did not hold the wiring.
