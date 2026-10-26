```yaml
lens: bound-breaker
outcome: reported
```

Task: bound-breaker lens. Attack every bound, guard, guarantee and claim about existing code in the decision record.

**Claims attacked**

1. D9: once Beam-Await is Done, the beam's dobby is stopped "so it starts no new member". **Broken.**
2. D9: the hold is rebased on "past Beam-Await", read from the run status. **Broken.** The board status the readers use can lag.
3. D9: re-run the land checks, then write the snapshot, remove the request and stop the dobby. **Broken.** There is a race between the re-check and the hold taking effect.
4. D9: at Beam-Await, members can fork, verbs run, and the entry stays in the Project layer and stays editable. **Broken.** A task cannot be made a member of a run-held project at all.
5. D10: the pawl run is "driven on by the parent's dobby". **Broken** for the parent's capacity: there is no bound on `max_runs`.
6. D4: "`runproc` gains that role-scoped end if it lacks one". **Broken.** This is a deferred fact, and it can be checked today.
7. D4 bound: a hand `lyx batten run` restarts the dobby. **Held**, since `ensureParentDobby` restarts the parent dobby. The member it starts is refused at Worktree-Create by D9's hold, which holds unless claim 2 bites.
8. D7 bound: the verdict is as fresh as the local Bolt. **Held** as stated. The direction on a failed owner read is not stated (finding 7).
9. D9 bound: a batten pass may re-spawn the driver once per probe pass, each one yielding, until the owner pause stands. **Held.**
10. D9: a land request is bound to the seed id, and Beam-Up and the owner yield remove stale requests. **Held** across cancel and re-create, since a new seed means a new id.
11. D5 upgrade bound: an older build writes task owners and the new build drops them on load. **Held.**
12. D5: `Effective` with no root owner reads a recorded beam owner. **Held.** D10 claims the root first.
13. D6: `lyx adopt root` reports a nested beam's pawl run as owned elsewhere. **Held.** D11 has the pawl run's verdict read the beam's own owner.
14. D11: cancel of a beam with no members ends the pawl run's processes and the beam's dobby and orch. **Held.**
    - The dobby is registered under the beam slug (`internal/beamcli/dobbydeps.go:65`), and `End(slug)` reaches it.
    - The orch strand is stopped by `stopPairStrands` (`internal/pairteardown/cancel.go:214`).
15. D12: one notice per episode, re-armed when a member is opened. **Partly unbounded.** D9 stops the dobby and `lyx beam start` restarts it, but where the episode is recorded across a dobby restart is not stated (finding 8).
16. D1 `--all`: one failure leaves the others started. **Held.**
17. D2: a recorded session that does not come up live falls back to a fresh start. **Held** as stated.

**Findings**

1. **BLOCKING, design, D9 last bullet.** At Beam-Await the beam cannot take a new member, so a beam lives for weeks without being able to grow.
   - `checkProjectField` refuses setting `project` to a project that holds a run status: `internal/boardengine/project.go:102`, "is held by a run".
   - The pawl run holds a run status for the beam's whole life, so this refusal stands the whole time.
   - D9's list of what Beam-Await does not hold (fork precondition, `ResolveBattenPrime`, D3, Project layer, run lock) leaves out board membership.
   - The rule should be stated over every `IsRunStatus` reader of a pawl-held entry, not as a fixed list.
2. **BLOCKING, design, D10 / D11.** Each pawl run waits at Beam-Await for weeks and occupies one of the parent dobby's `max_runs` slots.
   - Dobby counts every run it drives that is not done against `max_runs`: `internal/dobbyengine/doc.go:58`.
   - `max_runs` defaults to 3: `internal/dobbyengine/template.yaml:1`.
   - Counterexample: three beams under root on the default fill the prime dobby's slots, and root's own tasks never start.
   - No decision exempts a pawl run at Beam-Await from the count or states the bound.
3. **MEDIUM, design, D9 ("stops the beam's dobby so it starts no new member").** Every existing `dobbyengine.EnsureLive` caller restarts every owned beam's dobby, including a beam already past Beam-Await.
   - The callers are adopt's `StartDobbies` (`internal/adoptcli/deps.go:122`) and the anchor move (`internal/fabriccli/anchordeps.go:272`).
   - D6 makes adopt run the beam steps per adopted project but leaves this all-beams restart in place.
   - Counterexample: `lyx adopt X` during G's Parent-Align restarts G's dobby. It starts released members, each refused at Worktree-Create, and the refusals churn.
   - D9 states no bound for this.
4. **MEDIUM, design, D9 accepted-request order.** There is a window between the re-run of `lyx beam land`'s checks and the hold taking effect.
   - In that window the dobby is still live, or a hand `lyx batten run <member>` is running, and the board status still reads `· Beam-Await`.
   - A member released in the window passes `beamForkPrecondition` (`internal/battencli/beamfork.go:62`) and forks from a beam that then aligns, reviews and lands.
   - No row re-checks the members after Beam-Await. Finalize and teardown then run with a live member pair.
   - The fix is to stop the dobby and make the hold effective before the re-check, or to re-check at Finalize.
5. **MEDIUM, design, D9 / D3 "past Beam-Await".** The hold readers read the board entry's run status, and the board status write only warns on failure.
   - The readers: `beamLanding` (`internal/battencli/refusal.go:100`), `beamForkPrecondition` (`internal/battencli/beamfork.go:62`), and the layer and run lock.
   - The write: `internal/loomcli/wiring.go:231-235`, "a failed board write only warns".
   - Counterexample: the Beam-Await → Parent-Align status write fails. The board keeps reading `running · Beam-Await` through the landing, so members fork, verbs run in the beam and the entry stays editable.
   - D9 neither names which signal "past Beam-Await" reads nor bounds a stale board.
6. **LOW, design, D4 ("runproc gains that role-scoped end if it lacks one").** This defers a fact the record could settle now.
   - `runproc` has `End`, `EndSparing` (spares named roles) and `EndProcess` (one unregistered pid): `internal/runproc/end.go:56-70`. It has no role-scoped end.
   - Existing code finds the dobby record by role through `runproc.List`: `internal/fabriccli/anchordeps.go:156-166`.
   - The fix is to record that it lacks one and state the end it gains.
7. **LOW, design, D7.** The decision does not say what the watcher does when it cannot read the owner.
   - Dobby treats an unreadable owner as not owned elsewhere: `internal/dobbyengine/loop.go:236-249`.
   - D7 does not state the watcher's direction. A watcher that read a failed read as owned elsewhere would stop a healthy orch.
8. **LOW, design, D12 ("one notice per episode").** The episode memory has no stated home.
   - D9 stops the dobby, and `lyx beam start` or any `EnsureLive` restarts it, so an in-memory episode would re-send the notice after every restart.
   - The record names no record that survives a dobby restart.

**Tool calls**

1. Grep for `EnsureLive` and `DobbyStart` callers outside `dobbyengine`. The manifest did not cover which paths restart a stopped beam dobby (finding 3).
2. Read `battencli`'s `ensureDobby`. The manifest did not cover which dobby a hand `lyx batten run` restarts (claim 7).
