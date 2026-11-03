```yaml
lens: code-affected-set
outcome: reported
```

**Lens: code-affected-set.** For each change the diff makes, here is what depends on it outside the lines it touches, and whether the diff covers it.

Per change:
- **Hidden `lym hibernate watch` verb, `Watch`, `spawnDaemon`/`killDaemon`, `watch.log`, `Deps.Spawn`/`Kill`, `CancelReport.DaemonStopped` removed.**
  - Set found: a repo-wide grep over `.go`, `.md`, `.yaml` and `.json` files for `hibernate watch`, `daemon_stopped`, `hibernateengine.Watch` and `watch daemon`.
  - Result: no live reference to any of them remains. Covered.
  - One stale mention remains, in a test comment (finding 1).
- **`Submit` now returns `SubmitReport` and takes no `Spawn`.**
  - Set found: the CLI request verb, the `Deps` fillers, `verbs_test.go`, `world_test.go`, the hibernatecli integration test, and the `hibernatecli` and `hibernateengine` docs.
  - All are in the diff. Covered.
- **Request file now carries `daemon_pid` for a dobby.**
  - Reader outside the engine: `fabriccli/anchordeps.go`, which decodes `daemon_pid`, `daemon_start` and the added `phase`; both its wording and its `anchor.go` text are updated.
  - Covered in code and docs. Test coverage is not my lens (note 2).
- **Request file added to the prime dobby's watch set.**
  - Set found: `dobbyWatchDirs`, its callers and `beamcli/doc.go`. Covered.
  - Gap: finding 2.
- **Prime-only `Deps.Tick` and wrapped `CheckBinary`.**
  - Set found: `dobbyengine/loop.go:602` is the only consumer, and `beamcli/doc.go` carries the wiring.
  - `internal/dobbyengine/doc.go` says nothing of a Tick hook or a hibernate carrier, but the hook is a seam of `beamcli`, so I count it covered.
- **Docs and rule documents.**
  - `docs/overview.md` (module table and hibernate section), `PATTERN-daemon-wakeups.md` (the trigger-wait entry), and the `hibernatecli`, `hibernateengine`, `beamcli`, `fabriccli`, `cleanupcli` and `lymdirs` doc comments are updated.
  - `contracts/orchrole/orch-role.md:290-294` describes `request`, `status` and `cancel` without the daemon, so it stays correct.
  - No other PATTERN entry names the hibernate daemon: I found only a config-layers list and the denied `systemctl` forms.
  - Covered.
- **Declared PATTERN changes.** The directive's constraints section lists no declared changes, so there is nothing to land.
- **Commit message.** One commit, no `Pattern-Departure:` trailer, and it names no departure. Nothing to judge.

Findings:

1. **NIT, class `consistency`.** `internal/cleanupcli/cleanup_integration_test.go:329` still reads "a pair the hibernate daemon paused". The same diff reworded the cleanupcli docs from "daemon" to "the hibernate request's wake", so this comment was missed. Evidence: grep hit at that line.

2. **LOW, class `design`.** `internal/beamcli/dobbywatch.go:130` registers the watch on `filepath.Dir(hibernateRequestPath)`, the `hibernate` directory.
   - Nothing in the diff creates that directory at dobby start. The old spawn created it only for the log, and that spawn is removed.
   - On a hub that has never hibernated, the request write is the first thing to create it. The watcher for it opens only on a later event, because `openDobbyWatch` and `sync` skip a missing directory (`dobbywatch.go:76-98`).
   - Until then the prime dobby takes the request up on its timer. The idle ceiling is one minute (`loop.go:27`).
   - This contradicts the summary's "within a second of the write" and the `beamcli/doc.go` claim that a written request wakes the tick.
   - The request is still taken up, so this is delay rather than loss.

Notes outside my lens:
- (a) The new phase-skip in `fabriccli/anchordeps.go` `hibernateDaemonLive` appears to have no test of its own; `anchor_test.go` only fakes `HibernateLive`. This is for the tests lens.
- (b) Cancel then a fast new submit can leave the old goroutine in `waitForTrigger`, whose sleep is up to 30 s. `Carrier.running` stays true until then, so the new request waits. This is for the correctness lens.

Tool calls:
1. Grep across the repo for stale daemon, watch and spawn references in `.md`, `.go`, `.yaml` and `.json` files, because my context held only the diff.
2. Grep of `PATTERN.md`, `pattern/` and `docs/` for hibernate mentions, to find rule documents that describe the changed behaviour.
3. Grep of `dobbyengine`, `beamcli`, `fabriccli` and `cmd/lym` for hibernate, plus the `HibernateLive` and `hibernateDaemonLive` callers, to find consumers.
4. Grep of `orch-role.md` and the fabriccli tests for hibernate, and of `hibernateengine` for leftover "daemon" wording, to check the stencil and the tests.
5. Grep of `dobbyengine/doc.go` and `docs/overview.md` for Tick and prime mentions, to check whether the dobby docs needed an update.
