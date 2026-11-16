```yaml
lens: bound-breaker
outcome: reported
```

Task: attack the record's bounds, guarantees and code claims (bound-breaker lens).

**Findings**

1. **MEDIUM — D7, the upgrade's kept-cancel write.** The first dobby start after the upgrade can turn a landed run's leftover pair into a kept cancel.
   - The rule writes a finished cancel with keep for every pair whose slug has an unclaimed board entry, no batten record and no live process.
   - A landed run whose done-run clear deleted its batten record matches all three. Its entry is `done` and holds no claim, and its leftover pair is exactly what D7's first bullet tears down.
   - The FSM puts `landed` first, but `landed` needs "no batten record" when the batten record isn't `done`. After this write the run reads `cancelled` with keep instead.
   - The pair is then listed as kept forever and never torn down. That contradicts the D7 criterion that a landed run's leftover pair is torn down.
   - Fix: require the board status to be empty, not `done`, before writing the kept-cancel record.

2. **MEDIUM — D9's pause step and its first criterion.** The stop's pause step breaks when a hub pause already stands.
   - `internal/tenderengine/pause.go` reads: "A scope that already had a live entry adds none". The report then carries `AlreadyPaused`.
   - When the operator's hub pause or hibernate's entry stands, the stop owns no entry, so step 6 has nothing to lift. The other entry rightly survives.
   - The criterion "leaves no pause entry" is then false. The record also doesn't say that runs stay paused after the restart in that case.
   - Fix: state that the stop leaves a pre-existing hub entry standing, and narrow the criterion to "its own entry".

3. **MEDIUM — D10's hub skip.** Skipping a whole hub whose prime another machine owns breaks "starts only what this machine owns" in the other direction.
   - A beam this machine owns, inside a hub whose prime another machine owns, is never started.
   - `dobbyengine.OwnedWorktrees` would return that beam: the subagent reported that it filters beams by owner independently of the prime.
   - The launcher-removal rule ("finds the hub owned by another machine") would also delete the launcher this machine still needs for that beam.
   - Fix: skip per worktree, not per hub. Remove a hub's launcher only when this machine owns nothing in the hub.

4. **LOW — D9, `orchcli.BeamDown` gaining the Claude-process end.** This reaches more than `lym beam stop`.
   - `BeamDown` is also called by `internal/battencli/cancelcmd.go`, the cancel of a beam, and by `internal/loomcli/pawlwiring.go`, pawl's beam-down row.
   - Ending the beam's orch Claude there is probably wanted, but the record names only `lym beam stop`.
   - An orch agent running `lym beam stop` on its own worktree would end its own Claude process while the command is still running.
   - Fix: name those callers and bound the self-stop case.

5. **LOW — D4, hand-pass absence.** The Sketch scoped a hand pass's catch-up to "its run". The record dropped that, so every hand pass now fetches every local pair's branches (step 2).
   - That cost is unbounded and probably unintended.
   - Fix: keep "over its run" for a hand pass.

6. **LOW — D9's button.** A hub that sets `buttons.prime` in its own layer replaces the group whole, so the default `stop` entry never shows there.
   - The record says the stop button "shows only in the prime orch's session" with no such caveat.

**Claims attacked**

- D7: the stranded-run condition takes neither a cancelled nor a landed run. Held: `strandedEnded` requires `EntryEnded` and `State != done`.
- D9: idle confirmation within "hibernate's own wait". Held: `awaitConfirmation` is bounded by `confirmForkRereads`.
- D9: "leaves no pause entry". Broken (finding 2).
- D9: `BeamDown`'s change reaches only `lym beam stop`. Broken (finding 4).
- D7: upgrade kept-cancel records never touch landed runs. Broken (finding 1).
- D10: "starts only what this machine owns". Broken for owned beams under a prime owned elsewhere (finding 3).
- D10: never claims a prime. Held, because a hub with no prime owner is skipped before `lym beam start --all` runs.
- D4: the catch-up bound for a hand pass. Weakened (finding 5).
- D1: offline cancel re-run, and D5: a slug with a live entry gets no `hold` entry. Held, as far as my context reaches.
- D9: the button shows only in the prime session. Holds only for hubs that don't override `buttons.prime` (finding 6).

**Tool calls**

1. Grepped `BeamDown(` callers and `awaitConfirmation`'s loop bound. My context lacked both `BeamDown`'s callers and whether hibernate's wait is bounded.
2. Read `strandedEnded` in `internal/dobbyengine/ended.go` and grepped `boardengine` for claim clearing on `done`. I needed to test D7's claim about today's teardown condition.
3. Grepped `tenderengine` and `pauserecord` for how an existing live hub entry is handled. I needed to test D9's own-entry assumption.
