```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: I attacked the record's bounds, guards, ordering promises and claims about existing code, and built a counterexample wherever one exists.

## Findings

1. **BLOCKING — D4 / D8 step 2: the adopter can yield to the old owner.** D4 never says the takeover push is exempt from its own rule.
   - Counterexample: B writes the hold and owner in step 1. Old owner A keeps driving until its next Bolt pull, which can take up to 5 minutes. In that window A pushes a transition on `main-weft` or on a pair's records branch.
   - B's takeover push in step 2 is then rejected. B fetches and finds a commit whose trailer names A. Under D4, B stops acting on the run, archives its tips and resets to origin. The adopter yields to the old owner, and A is never stopped by any takeover commit.
   - The same window exists if the adopt is killed between steps 1 and 2. A acts on runs that Bolt says B owns, and nothing rejects it until A's dobby pulls Bolt.
   - D4 needs a rule that a takeover push replays over the old owner's commits. It also needs a bound for the window between the owner write and the takeover.

2. **BLOCKING — D4: "a commit by another machine means another machine acted on the run" is false.** The design itself makes non-owner machines write to run records branches.
   - D3 has tender write the pause intent and the cancel intent into the run's tracked record and push them synchronously. An operator on machine B runs `lym tender pause <slug>` for a run that A owns. That pushes B's commit, and A's next transition push is rejected over it. D4 then makes A stop the run, archive it and reset, so a pause has become a takeover.
   - D7's upgrade conversion behaves the same way. It turns a standing adopt entry into a pause intent on whichever machine still holds that entry, and that machine may no longer own the run.
   - `main-weft` makes it worse, because it holds every root run's batten record. One foreign commit there, such as a pause, a cancel, or a worktree-layer `lym config` on the prime, rejects the owner's pushes for all root runs at once.
   - The rule needs to tell takeover commits apart, for example with a takeover trailer, rather than treating any foreign machine name as a takeover.

3. **MEDIUM — D4 upgrade: commits without a trailer have no rule.** An older lym still running on the other machine, or records commits made before the change, carry no `Lym-Machine` trailer. A rejection over such commits fits neither "names another machine" nor "all name this machine".

4. **BLOCKING — D9: the A6/A7 holder is neither of the two processes D9 is allowed to end.**
   - D9 ends a holder only if `internal/runproc` registers it for the worktree's orch, or if it is the pane process of the orch strand.
   - The orch's Claude process is not registered in runproc. `RoleSession` is registered only for run-scoped sessions (`loomcli/sharedbootstrap.go`), and the orch packages register no session.
   - In A6/A7 the strand was dead or removed while the Claude process lived on. The watcher's `RemoveStrand` does this, and so does `BeamUp`'s relaunch through `RemoveStrand`. So there is no strand whose pane process could match.
   - Under D9 that holder counts as "not owned by lym". The orch start refuses and the session stays held. That is exactly the case the decision claims to fix.
   - The orch's Claude process must be registered in runproc when it launches, and that registration must outlive strand removal.

5. **MEDIUM — D4 bound, D6 step 4, D11: uncommitted work is neither archived nor removable.**
   - D4's bound says the old owner's in-flight result "is archived under a tag, not lost". `ArchiveOwnerTips` and `DropLocal` archive commits only; the state map (section 4.3) says "uncommitted files are never archived".
   - D6 step 4 removes the pair of a run owned elsewhere. A pair whose agent was mid-edit is dirty, so the teardown refuses with `worktree has uncommitted changes; use --force`. That is A12 again.
   - D11's orphan teardown hits the same refusal on a pair the operator hand-edited without committing.
   - The record needs to say what happens: commit the files onto the archive, refuse and post a notice, or remove with force.

6. **MEDIUM — D6 / D1: a hand pass after an absence skips the catch-up.** The three absence triggers are an observed sleep, a dobby start and a rejected push.
   - Counterexample: the machine is off, then the operator types `lym batten run <slug>` before any dobby has started. No catch-up runs, and the act rule reads a stale local Bolt and answers `act`.
   - The first producer then starts an agent step on a run another machine took. D2 pushes only after the transition, so nothing stops the step from running.
   - The origin-owed marker retires, and the record never says that a hand pass keeps the origin reconcile `PATTERN-owner-machine` gives it today.

7. **MEDIUM — D7: hibernate's pause becomes a Bolt hold.** D7 says hibernate stays machine-local, but hibernate pauses through tender: `hibernateengine/request.go` holds `Pause func(tenderengine.PauseRequest)`, and `wake.go` builds tender pause requests.
   - D7 makes a hub-wide `lym tender pause` write a Bolt `pause` hold. A hibernate on A would then push a hub-wide hold that stops B from acting on the beams it owns.
   - The hold could also be left standing when A never wakes. Hibernate needs a local-only path, named in D7.

8. **MEDIUM — D7 / D8: the hold covers only the subjects, but the operator's rule covers the whole hub.**
   - The board says that until adopt is done and its failing tests are fixed, "no run starts, no dobby pass, no orch action" happens in the hub.
   - D7's hold covers "the subjects it covers". With `lym adopt <beam>`, root's runs and other beams' runs on the same machine keep running while the hold stands and the fix task runs.
   - D8's rationale still claims the operator's rule is met.

9. **MEDIUM — D8: the `--release` guard can be bypassed.** The guard is that the caller's strand name is set, which is the `LYM_STRAND_NAME` env var.
   - Any agent's Bash can run `env -u LYM_STRAND_NAME lym adopt --release <subject>`. So "only an operator's shell runs it" does not hold.
   - The bound must either state that it is bypassable or add a shell-guard deny row.

10. **MEDIUM — D10: "more of the transcript" by length does not pick the right copy.**
    - Counterexample: before D9 ended it, A's stale orch process appended turns to its local copy of the same session id while B continued the session. The two files now diverge.
    - Whichever copy is longer wins. A long stale copy is kept locally, or a long stale mirror overwrites a shorter, newer local continuation.
    - Comparing the last entry's uuid or a common prefix would separate "ahead" from "diverged".

11. **MEDIUM — D10: the mirror's force-push has no owner check.** The mirror runs from the orch's Stop hook at turn end, and the act rule and catch-up do not guard it.
    - Counterexample: A wakes from sleep before its catch-up has ended its orch. A queued creel mail triggers a turn, and the turn end force-pushes A's stale transcript over B's mirror.
    - A later restore on a third machine, or on B after a crash, gets the stale transcript.
    - The mirror should push only when this machine owns the subject, or push with a lease on the ref's old value.

12. **MEDIUM — D1 / D2: `landed` is derived from a board status that D2 lets lag.**
    - D1 derives `queued` and `landed` from the board status. It also calls that status "a display, written from the FSM state", which is circular.
    - D2 keeps display-only board writes on the detached sync, and the `done` write on landing is not on D2's list of synchronous decisions.
    - Counterexample: a landing's `done` write lags. Another machine reads `landing`, not `landed`, gets `act` from the act rule, and runs Finalize again.
    - D2 must list the `done` write as a decision.

13. **LOW — D11: a kept pair can be torn down as an orphan.** The adopt fix loop's own hand steps are `lym batten cancel --remove` followed by `lym board remove`. An operator who cancels a run while keeping its pair and then removes the entry leaves a pair with no board entry. D11 tears that pair down after the orphan grace, against the operator's keep.

14. **LOW — D5: former names on record before the upgrade never reach the registry.** D5 seeds the registry only from future `lym machine name` calls and first owner writes. If registering does not upload `machine.json`'s existing `names[]`, every owner written as `osl-1046` before the upgrade still reads as a different machine from `osl46`.
    - A name freed by a rename and reused by another machine would also be aliased to the wrong machine.

15. **LOW — D3, criterion "opens no second pull request".** The tracked pull-request record reaches origin only at the next transition, so a pull request opened mid-step is not on origin yet. The criterion holds through `FindOpen`'s search of GitHub, not through the record. Per the exploration, a "yes" landing that is not MergeAtOnce and has no record halts naming `lym loom goto PR-Review`.

## Claims attacked

- D2, "the next producer starts only after the push lands", including a held network gate under a pause: held.
- D2's bound, "no run acts offline": held for transitions. It is broken for a hand pass that starts a step on stale state (finding 6).
- D4, "the old owner's next push is rejected": broken by the step 1 to step 2 window (finding 1).
- D4, "a commit by another machine means it acted": broken (finding 2).
- D4's trailer classification: broken for old commits (finding 3).
- D4's bound, "the result is archived, not lost": broken for uncommitted files (finding 5).
- D5, "former names make one identity": partly held (finding 14).
- D6's absence triggers: broken for a hand pass (finding 6).
- D6 step 3, "push owed commits": held.
- D6 step 4, "remove the pair": broken for a dirty pair (finding 5).
- D7, "hibernate stays local": broken (finding 7).
- D7's upgrade conversion: broken for a non-owner machine (finding 2).
- D8 step 1, the race between two adopters: held.
- D8, "nothing happens in the hub": broken (finding 8).
- D8, "re-run is idempotent": held for each step alone. It is broken for the window between steps 1 and 2 (finding 1).
- D8, `--release` refused from agents: bypassable (finding 9).
- D9, "ends this worktree's own holder": broken for A6/A7 (finding 4).
- D9, the claim about the existing fresh-launch fallback in `orchcli/start.go`: held.
- D10, "restore when origin holds more": broken (finding 10).
- D10's mirror safety: broken (finding 11).
- D10's chunk-delta claim: held.
- D11, "kept pairs stay": broken by entry removal (finding 13).
- D11, `rerun`'s failure ends because tender pushes no pair: held.
- D1, the FSM is derived from tracked records: broken for `landed` (finding 12).

## Tool calls

1. Read `internal/runproc/record.go`'s roles and searched `orchcli`/`orchengine` for runproc use, to learn whether the orch's Claude process is registered. My context only had the subagent summary.
2. Searched `internal/hibernateengine` for its pause path and `agentname` for the strand env var's name, neither of which my context held.
3. Searched for `RoleSession` registrations, to learn which sessions runproc registers.
