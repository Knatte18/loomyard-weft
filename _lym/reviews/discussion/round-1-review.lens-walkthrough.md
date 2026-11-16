```yaml
lens: walkthrough
outcome: reported
```

I walked each decision end to end on one concrete case and checked the end state against the decision's purpose.

**Decisions walked:**
- D1, a task-pair transition with origin unreachable: the push is owed, the act rule waits on `PushOwedOffline`, and the catch-up's step 3 pushes the local commits after a process dies. Purpose met.
- D2, an adopter's takeover landing on the old owner's next records push: fetch, read the takeover entry, hand over, archive. Purpose met, but the "has not yet read" state is unnamed (finding 4).
- D3, a machine `osl46` renamed to `osl-1046`, with Neve comparing an owner of `osl46`. Purpose met.
- D4, a dobby sleep followed by another machine's takeover, then catch-up steps 1 to 5. Purpose met.
- D5, a hold written by B, read by A at its Bolt pull: A's runs in the subject answer `hand_over`, its other runs get `hold` entries, and the entries drop once the hold clears. Purpose met. The upgrade's first read misses its purpose (finding 2).
- D6, a re-run of `lym adopt X` on the adopting machine after a failing check left an `adopt-fix` hold. Purpose missed (finding 1).
- D7, a cancelled run kept before the upgrade, then the first dobby start. Purpose met.
- D8, `lym board get` of an entry under the prime, read by the orch role. Purpose met.
- D9, a stop clicked in the prime session: pause, end, lift by identity, deferred pushes paid. Purpose met.
- D10, a machine that owns only a beam in a hub whose prime another machine owns. Purpose missed (finding 3).

**Findings:**

1. **MEDIUM, design, D6 steps 1 and 8.** A plain re-run of `lym adopt` under this machine's own `adopt-fix` hold loses the fix task's exemption and the try counts.
   - Case: the check failed, the hold is `adopt-fix` naming the fix task with try count 1, and the operator re-runs `lym adopt X`. D6 says "a re-run goes on from where the state stands", and D5's upgrade notice names this re-run.
   - Step 1 refuses only "a hold another machine wrote". For this machine's own hold it "writes an `adopt` hold", replacing the `adopt-fix` hold and its try counts.
   - D5 sets `Held` false only for "the `adopt-fix` hold's fix task", so under the new `adopt` hold the open fix task's run is held.
   - At step 8 a failing check reaches the fix loop with an open own fix task and a reset count.
   - Only `--take-hold` states "keeping kind, fix task and try counts".
   - The same gap applies to the upgrade. Today the try count lives in a local tries file (`internal/adoptcli/holds.go:21-31`, written at `internal/adoptcli/fixloop.go:279`). D5's upgrade writes only an `adopt` hold, which neither carries that count nor exempts a fix task already open.
   - Evidence: `decision-record.md:189`, `:200-203`, `:208`, `:151`, `:171`.

2. **MEDIUM, design, D5 upgrade (`decision-record.md:169-172`) against D6 step 8 (`:199`).** Following the upgrade notice's way forward leaves the runs paused.
   - The first read turns each standing `adopt` entry into a run pause intent and posts a notice whose only way forward is "a re-run of `lym adopt` of those subjects".
   - The re-run's green check clears the hold, and "the runs go on by the act rule".
   - The act rule still answers `wait` with reason `paused` for each run, because the pause intent stands (`internal/runfsm/act.go:97-98`, `:213`).
   - The notice never names `lym tender resume &lt;slug&gt;`, so the end state misses D6's "runs go on", and nothing tells the operator.

3. **MEDIUM, design, D10 launchers (`decision-record.md:313`).** A machine that owns only beams in a hub never gets a launcher.
   - D10 writes a hub's launcher, and the all-hubs launcher with the first hub, only "when this machine's prime dobby starts".
   - Case: B owns the prime and this machine owns beam X. This machine's prime dobby never starts: `OwnedWorktrees` omits a prime another machine owns (`internal/dobbyengine/start.go:105`), and D10 skips it.
   - So no launcher is ever written for that hub, although D10's own verb would start X there, and the removal rule keeps a launcher "while this machine owns anything".
   - The board entry's purpose, "brings back every hub and beam that was in flight", is missed for beams owned under a prime another machine owns. Criterion `decision-record.md:410` restates the prime-dobby trigger.

4. **LOW, design, D2 hand-over trigger (`decision-record.md:73`).** "A takeover entry naming another machine that this machine has not yet read" depends on a per-machine record of takeover entries already read. No decision names that record, says where it lives or who writes it.
   - The walk hits it right away: after a hand-over the old owner's local pair stays until a catch-up removes it (D4 step 4, `:131-133`). Every rejected push or fetch before then reads the same entry again.
   - It is not the D2 bullet's to say whether the hand-over then runs again, ending processes and pushing archive tags again, or is skipped by a mark that lives somewhere.

Tool calls:
- grep of `internal/adoptcli/steps.go`: to see whether today's adopt steps add the subject worktree; the manifest had none of adopt's steps.
- Read of `internal/tenderengine/resume.go:225-314`: to see what D9's `Lift` spawns after the stop; the manifest held only `liftEntries`' head.
- Read of `internal/dobbyengine/start.go:80-179`: for `OwnedWorktrees` and `EnsureLive`, which D10 filters and the manifest only located.
- grep of `internal/tendercli/runintents.go` and the adopt tries helpers: for where the fix-try count lives today, needed for finding 1's walk.
