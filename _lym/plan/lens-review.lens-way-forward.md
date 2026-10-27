```yaml
lens: way-forward
outcome: reported
```

Way-forward lens: each refusal, stop and recovery step the cards add, the next step it names, and whether that step can succeed from the state it leaves.

**Findings**

1. MEDIUM — Card 10 (Decision 3, the `placeDobby` refusal after a re-exec): the notice lacks "once its cause is fixed", so the named next step repeats the refusal.
   Card 10 step 1 posts `<worktree>: the dobby could not start after a re-exec: <error>` and leans on the refusal's own text for the way forward.
   That text is `placeDobby`'s `refuse` format, `way forward: run \`lym beam start\`, which starts the dobby…` (`internal/beamcli/cli.go`, `placeDobby`), with no "fix the cause".
   Decision 3 says each of the other three exits "names the error and `lym beam start` once its cause is fixed".
   For a refusal at `RequireDrivableWorktree`, `PrimeName`, `ReadOrigin`, the prime's resolve or `hubgeom.DobbyGeometry`, an immediate `lym beam start` spawns a dobby that hits the same refusal.
   Card 10's own wiring-failure notice (step 3) does say "fix the cause, then run lym beam start"; the refusal notice should say the same.

2. MEDIUM — Cards 14 and 15 (Decisions 10 and 11): a `check.json` that does not decode blocks every recovery.
   Card 14's `ensureCheck` reads `check.json` through `ReadCheck` (card 13: `state.ReadJSON`), and the card says nothing about a decode error.
   If that error comes back as `ensureCheck`'s error, every later `lym adopt` gets the warning and spawns no check.
   That warning names `lym fabric reconcile` and `lym gate test --gate adopt`; neither touches `check.json`, so the next adopt fails the same way.
   Only a `runCheck` could write a fresh record, and none is spawned.
   Card 15's `ReadCheckView` joins the same error into `dobbyStatus`, whose way forward, `fix the cause each names, then run lym beam status again`, never says to remove the file.
   The precedent is `dobbyengine.ReadState` (`internal/dobbyengine/state.go`), which reads an undecodable record as absent with one Warn.
   The cards should prescribe that for `ReadCheck`, or name the file's removal as the way forward.

3. MEDIUM — Card 7 (Decision 3, a fatal error when the state read fails): the line and its ending contradict each other.
   Step 1 says a `ReadsAsReexec` read error "keeps its line only".
   Step 5 writes the stop line only for "a non-nil error out of `Run` after step 1", which leaves the step-1 error with no line.
   Decision 3 and acceptance criterion "each exit path of Decision 3 writes its line" need a line there.
   No test row reaches this branch.

4. MEDIUM — Card 10 step 4 (Decision 5): `dobby.out` rotates before the lock is held.
   Step 4 rotates `dobby.out` and reopens stdio "and then" after setting `deps.Locked`, still before step 5 runs `Run`, and `Run` is what takes the lock.
   As written, a process that will lose the lock renames the winner's `dobby.out` to `.1`.
   That breaks "only the lock holder rotates `dobby.out`" (the overview's Shared Decisions and Decision 5).
   The rotation belongs inside the `Locked` callback.
   This is a correctness gap outside this lens, but the refusal path is where it shows.

5. LOW — Card 7 (Decision 2, the refused-re-exec notice): `lym beam start` does nothing while the lock is still held.
   The notice names `lym beam start`, which reaches `ensureOne` → `Live` → `OutcomeLive` (`internal/dobbyengine/start.go`).
   So it starts nothing while the holder that kept the lock past ten seconds still holds it, and reports the dobby live although no dobby recorded itself.
   It succeeds only once that holder releases.
   The notice could say to re-run `lym beam start` until it reports the dobby started, and `lym beam status` to check.

6. LOW — Card 3 (Decision 8): a failed machine-name read in `Landings` stops the rest of every tick.
   The error becomes a tick failure, and `countLandings` returning it skips `tidyStep`, `landNotice` and the `Tick` hook on every tick.
   That holds the "can land" notice and the tidy back.
   The episode's line, `tick failing: <error>`, names no way forward such as giving the machine a hostname.

7. LOW — Card 15 (Decision 11): "ended without a verdict" can never clear.
   It names the rerun `lym gate test --gate adopt`, which writes no `check.json`.
   So `lym beam status` shows that state until a later `lym adopt` with an owner write spawns a fresh check.
   The view could also name that re-adopt.

8. LOW — Card 7 (Decision 3, the owner exit): the line names no next step.
   `stop: another machine owns this beam (<owner>)` could name `lym adopt <subject>` to take the beam back, as `EnsureLive`'s refusal does (`owned by %s; lym adopt %s takes it over here`, `internal/dobbyengine/start.go`).

**Per refusal: the next step, and whether it succeeds**

- Card 2, `BuildShed` with no machine name → give the machine a hostname, then `lym batten run <slug>`: succeeds.
- Card 7, refused fresh start → nothing named, which is right (the idempotent case): succeeds.
- Card 7, refused re-exec → `lym beam start`: succeeds only after the lock frees (finding 5).
- Card 7, fatal error after a re-exec, including a `Locked` error → fix the cause, then `lym beam start`: succeeds, since the lock is released and `EnsureLive` spawns.
- Card 7, failed exec → fix the binary, then `lym beam stop` and `lym beam start`: succeeds; `beamDown` ends a live dobby (`internal/orchcli/beamdown_test.go`), and a fresh binary identity is also re-exec'd on its own.
- Card 8, failed tidy stop → `lym reed remove <guid>`: succeeds; the verb exists (`internal/reedcli/remove.go:118`).
- Card 10, wiring failure after a re-exec → fix the cause, then `lym beam start`: succeeds.
- Card 10, `placeDobby` refusal after a re-exec → `lym beam start`: does not succeed until the cause is fixed (finding 1).
- Card 13, reconcile failure → `lym fabric reconcile` in the prime, then `lym gate test --gate adopt`: succeeds; `runReconcile` runs `configsync.ReconcileHubWideAt` with migrations (`internal/fabriccli/fabric.go:708`).
- Card 13, failed check → rerun `lym gate test --gate adopt` in the prime: succeeds for the verdict, though no record is updated (finding 7).
- Card 13, check lock held → exits 0, nothing needed: succeeds.
- Card 14, check not started → `lym fabric reconcile`, then `lym gate test --gate adopt`: succeeds, except over an undecodable `check.json` (finding 2).
- Card 6, failed Windows rotation → left for the next spawn: succeeds.

**Tool calls**

- Grepped `fabriccli`, `reedcli` and `orchcli` for the reconcile verb, `reed remove` and `beam stop`'s dobby end, to check that the named next steps exist; my context lacked those packages.
- Grepped for `hubreconcile` callers to see whether `lym fabric reconcile` runs the config reconcile; my context had only `beamcli`'s and the clone's.
- Listed `internal/fabriccli` reconcile files and grepped for `Migrate` and `configreg` callers, to find where the gate migration runs.
- Grepped `internal/fabriccli/fabric.go` for `configreg`, `Migrate` and `hubreconcile`, to locate `runReconcile`.
- Read `internal/fabriccli/fabric.go:708-760`, `runReconcile`, to confirm it runs the hub-wide config reconcile with migrations, which card 13's and card 14's ways forward depend on.
