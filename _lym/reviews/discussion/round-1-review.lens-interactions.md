```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I checked every pair of Decisions that acts on the same state, file, key or command, and checked the criteria and constraints against the Decisions they touch.

Pairs checked:
- D1 × D2 (graced lock in `Run`, wait picked by reading the state record first): held.
- D1 × D3 (lock loser prints the envelope and exits; the refused line and notice): held.
- D2 × D3 (a re-exec that loses takes the refused-re-exec path): broken, F3.
- D2 × D4 (second dobby wins, and the tidy in flight): held. `tidyStep` settles `State.Tidy` whatever the pid (`internal/dobbyengine/tidy.go:146-150`).
- D1 × D4 (re-exec reaches `Run`): held.
- D3 × D5 (exit lines go through the per-line seam; stdout moves to `dobby.out`): broken, F1 and F4.
- D3 × D6 (start line and start heartbeat; fatal-error line and the failing-pass episode): held.
- D5 × D6 (fewer lines and rotation): held.
- D5 × D2 (a re-exec spawns nothing, so `dobby.out` grows): held, and D5's bound says so.
- D6 × D2 (in-memory run states lost at re-exec): held. The start heartbeat names every state, so a change inside the exec gap still shows.
- D6 × PATTERN-daemon-wakeups (heartbeat rides the existing ticks): held. `idleCeiling` is one minute (`internal/dobbyengine/loop.go:23`).
- D7 × D8 (`machine` written, then read): held only partly, F5.
- D8 × adopt (a landing tidied once): held. The baseline and the machine filter keep another machine's landings out.
- D9 × D10 (the check reads the `adopt` key that the migration writes): broken, F2.
- D10 × D11 (`check.json` written by the check, read by beam status): broken, F6.
- D10 × D9 (rerun named as `--gate adopt`): held.
- D3/D10 × PATTERN-pane-text-injection (notices go through creel): held.
- D1 × the stop paths (`stopBeamDobby`, `lym beam stop`) with registration moved behind the lock: held. A re-exec keeps its pid, so the old image's record still matches until the new image re-registers.
- Acceptance criteria × Decisions: held for D1, D2, D3, D5, D7, D8, D9 and D11. Gaps for D6 and D10 in F7.
- Constraints × Decisions: held. selfexec stays a leaf, since the callbacks are plain funcs. The daemon-wakeups and gate-slot constraints hold too.

Findings:

F1. MEDIUM, design. D3 and D5 together lose the exits before `Run` from `dobby.log`.
After D5, stdout and stderr go to `dobby.out`. So a `placeDobby` refusal or a `hubDobbyDeps` wiring error leaves no line in `dobby.log` any more; today it does, through stdout. D3's list of exits covers only exits from `Run` onward. After a deploy, a new binary that refuses to place or wire the worktree ends the dobby, and the log the operator reads stays silent. That breaks the Goal's "never ends silently".
Evidence: `internal/beamcli/cli.go:373-386` (`output.Err` before `Run`) and `internal/dobbyengine/start.go:270-273` (stdout and stderr go to the log file).

F2. MEDIUM, design. D10's check can run before D9's migration has written `adopt`.
`lym adopt` runs only the origin reconcile, never `hubreconcile`. `loadGate` loads `gate.yaml` without reconciling. So the first adopt check after this binary deploys, before any start verb has reconciled the hub, reads the template's `adopt: {tiers: [0]}`. Today the same check runs `finalize`'s `[0, 1, 2]`. It goes green silently with fewer tiers. Neither Decision states this ordering, and D9's claim that "a hub's check keeps the tiers it runs today" does not hold in that window.
Evidence:
- `internal/adoptcli/deps.go:65` (Reconcile is the origin reconcile).
- `internal/gatecli/run.go:274-303` (`loadGate` calls no `hubreconcile`).
- `EnsureOrWarn` is called only from `preflightshed/preflight.go:96`, `loomcli/arm.go:153` and `battencli/arm.go:345`.
- `_board/_lym/config/gate.yaml` (`finalize: {tiers: [0, 1, 2]}`).

F3. LOW, consistency. D2 says the re-exec'd process exits "through Decision 3's refused-re-exec path", and that can be false.
The second dobby can take the lock and write its own pid through `recordStart` before the re-exec'd image reads the state record. The re-exec'd image then sees another pid and classifies itself as a fresh start. It waits only `lock.ProbeGrace` and exits on the refused-fresh path, which posts nothing. The hub still keeps one dobby, but the stated path and the criterion "posts the orch notice when it is a re-exec" both miss this case.
Evidence: `internal/dobbyengine/loop.go:190-203` (`recordStart` writes the pid on a fresh start), together with D2's rule of reading the record before acquiring.

F4. LOW, design. D5's rotation is not safe against D3's refused-start writer.
D3 has a process that does not hold the lock write its refused line to `dobby.log`, through the same rotating seam the live dobby uses. If both see the file over 1 MiB, both rename `dobby.log` to `dobby.log.1`. The second rename replaces the first's `.1`, which loses up to 1 MiB of history. D5 names no single writer for rotation.
Evidence: D5 (rename before a write that crosses the bound), D3 (the refused line), `internal/dobbyengine/loop.go:118` (only the holder of the lock is the dobby).

F5. LOW, design. D7 and D8 together let an unreadable machine name drop a landing for good.
D7 stamps `machine` only when the name is non-empty. D8 never counts an entry without one. `machinename.Name` can return an error, for example on an empty hostname. D7 does not say whether batten's wiring then refuses the run or tells an empty name. If it tells an empty name, that landing is never tidied on any machine.
Evidence: `internal/machinename/machinename.go:11-26`, D7 ("when the name is non-empty"), D8 ("never counted").

F6. MEDIUM, design. D10 and D11 give `check.json` two writers.
Each `lym adopt` call that records an owner spawns a check, so two adopts in a row run two checks. The record is "rewritten fresh at each start", so the second check overwrites the first. The first check then keeps writing its per-tier outcomes into a record that carries the second check's pid. It also writes into a `suite.log` the second check truncated. D11 then shows a mixed record, and both checks post per-tier lines. D10's only bound is that gate slots serialize the checks, and with `slots: 2` they do not.
Evidence: `internal/adoptcli/adopt.go:324-330` (one spawn per call that recorded an owner), `internal/adoptcli/deps.go:212` (`os.Create` truncates `suite.log`), and `slots: 2` in `internal/gateslot/template.yaml`.

F7. LOW, consistency. Two stated behaviours have no matching criterion.
- D6: a re-exec logs no change for a state it did not see change.
- D10: `suite.log` is truncated and the record rewritten fresh at each start.

Evidence: the Acceptance criteria section of `_lym/discussion/decision-record.md`.

Tool calls:
1. `internal/gatecli/run.go` `loadGate`, the `EnsureOrWarn` callers and `machinename.Name`: to learn whether a gate load reconciles, and whether the machine name can fail. The manifest had neither.
2. `adoptcli` reconcile wiring and `orchcli` `reconcileHub`: to learn whether `lym adopt` runs the config reconcile before the check. The manifest lacked this.
