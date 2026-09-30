# Support log — ly-drive-safe-repairs

## Interview

Autonomous session: no operator answered.
Each turn below is a question the design had to settle, answered by reading the codebase and the board, and then by an auto-pick recorded in the ledger.

1. **What does the driver do today at a stop?**
   The ly-drive skill hands back `blocked`, `paused` and `awaiting` unrepaired.
   Error envelopes of kind `producer`, `bootstrap` and `unseeded` already go down a trace-based repair path under a two-repair cap.
   The loom launch prompt ends every stop with `lyx loom commit-records; lyx reed remove --name loom-driver --detach`, so the session exits at every hand-back.
2. **How can the driver tell "transient" apart from a real stop?**
   The skill branches only on envelope fields and the five error kinds, never on text.
   Only the failure site knows transport from rejection, so the classification is Go-side: a typed mark in `shedengine`, surfaced as a new `transient` envelope key (D1, D2).
3. **Is a flaky-verify class needed?**
   `verify-triage` (in flight) reruns failing tests and compares with the parent, halting only on a real regression.
   After it lands, a flaky verify never stops the run, so a driver-side class would duplicate it with less information.
   Dropped from scope.
4. **How does the driver know "the lyx binary changed since the stop"?**
   `lyx` has no version verb, but the deployed binary carries `vcs.revision` and `vcs.modified` (checked with `go version -m`).
   Each step records the identity; `lyx shed status` derives `binary_changed` (D4).
   Recording the new identity at the re-step makes the bound "once per deployed binary" fall out without a counter.
5. **Which stops may a binary change re-step?**
   Not `awaiting` or `paused` (a person's hand-off), not `busy`/`ownership`/no-kind refusals, not an interrupted hand-back (live agent risk).
   `blocked` and escalated errors are eligible, guarded by a clean worktree (D5).
6. **How does a parked driver get woken?**
   `shuttleengine.Runner.Send` already types a verified single line into another process's run.
   Batten's `Spawn` and the hub both already run `lyx loom start`, so making `loom start` the resume route changes no caller (D8).
7. **How does `loom start` know the live driver is parked, not working?**
   The pane alone cannot tell: a driver waiting on a background step job is idle at its prompt too.
   A park marker file in the run's scratch directory gives a deterministic signal (D7).
8. **Where do stop reports go once a session spans several hand-backs?**
   Report paths are minted per attempt by Go; `loom start` mints a fresh one on resume and names it in the resume line.
   Self-initiated re-steps rewrite the attempt's file, with each version committed at the hand-back (D9).
9. **How does a parked driver watch for a deploy?**
   Polling `lyx shed status` writes a trace file per poll, and a long-lived `lyx` process blocks a Windows deploy's overwrite.
   A shell loop on the executable's mtime, confirmed by one status read, avoids both (D10).
10. **Does `done` still end the session?**
    Yes: batten's `Run-Shed` waits for a done child's driver to exit before teardown (`676d9ec03`).

## Rejected alternatives

- **Driver-side reason-text matching for transient stops.**
  Fragile against wording changes, and it breaks the skill's rule of branching only on fields and kinds.
- **A driver-side flaky-verify class.**
  Duplicates `verify-triage`, which has the test names and the parent comparison the driver lacks.
- **A new refusal kind (`transient`) instead of a key.**
  The Shed Verb-Set Invariant keeps the kind vocabulary closed at five, and a transient producer error is still a producer error for every other rule.
- **Build identity as a content hash of the executable.**
  Hashing a ~27 MB binary on every step costs I/O for no gain over the stamped revision on the production route.
- **A dedicated `lyx loom resume` verb, or the hub sending `lyx shuttle send` itself.**
  Every caller (hub, batten) already runs `lyx loom start`; a second verb would make each caller learn whether the driver parked.
- **Detecting "parked" from the stop report's existence.**
  A self-resumed driver keeps its report file, so the file cannot tell parked from working.
- **Detecting "parked" from pane readiness or `last_step.finished`.**
  A driver waiting on a background step job looks idle, so `loom start` would inject a resume mid-loop.
- **A park marker written through a new `lyx loom park` verb.**
  More CLI surface for a file the skill can write at a Go-declared name, pinned by a test.
- **Binary-change re-step on any stop, including `awaiting` and `paused`.**
  Both wait on a person by design; the body says `awaiting` is never auto-approved.
- **Polling `lyx shed status` as the watch.**
  Hundreds of trace files per overnight park, and more `lyx` invocations racing a deploy.
- **Keeping the exit-at-hand-back and relying on `loom start` to spawn fresh.**
  That is today's behaviour, and the `driver-restart-race` it produces is exactly what the body wants gone.

## Review rounds

_No rounds yet._

## Question ledger

- **Q1 — Does the flaky-verify class belong in this task?**
  Resolved, auto-pick: no; `verify-triage` owns flaky and pre-existing failures in Go.
- **Q2 — Where is "transient" decided?**
  Resolved, auto-pick: Go, as a typed mark in `internal/shedengine` translated at the producer and bootstrap boundary; surfaced as an envelope key.
- **Q3 — Which classes are transient?**
  Resolved, auto-pick: `git-transport`, `github-api` (transport or 5xx), `agent-start`; rate limits and every rejection are excluded.
- **Q4 — Transient re-step budget?**
  Resolved, auto-pick: one immediate re-step; a repeat stop at the same producer hands back; separate from the trace-repair cap.
- **Q5 — Build identity source?**
  Resolved, auto-pick: `vcs.revision` plus `vcs.modified`; unknown on either side means no re-step.
- **Q6 — Which stops are eligible for the binary-change re-step?**
  Resolved, auto-pick: `blocked` and escalated `producer`/`bootstrap`/`unseeded` errors, plus a failed transient re-step; never `awaiting`, `paused`, `busy`, `ownership`, no-kind, or an interrupted hand-back.
- **Q7 — Guard against re-stepping a worktree someone is fixing?**
  Resolved, auto-pick: skip while `git status --porcelain` shows any change; the hub can also `lyx loom pause` a run it is about to fix by hand.
- **Q8 — Resume channel?**
  Resolved, auto-pick: `lyx loom start` sends one line through shuttle `Send` when a live driver is parked.
- **Q9 — Parked signal?**
  Resolved, auto-pick: a marker file in the run's ephemeral scratch directory, filename declared through `internal/shedrun`, consumed by `loom start`.
- **Q10 — Which sessions park?**
  Resolved, auto-pick: loom-launched drivers only; orchestrator forks and operator-launched sessions keep today's stop behaviour and get only the immediate re-steps.
- **Q11 — Does `busy` park?**
  Resolved, auto-pick: no; another driver holds the run, so this one ends as at `done`.
- **Q12 — Report files across several hand-backs in one session?**
  Resolved, auto-pick: one file per attempt (spawn or `loom start` resume), rewritten after self-initiated re-steps, committed at each hand-back.
- **Q13 — Watch mechanism?**
  Resolved, auto-pick: background shell loop on the executable's mtime at about one minute, confirmed by one `lyx shed status` read.
- **Q14 — Does this task fix the `driver-restart-race` note?**
  Resolved, auto-pick: no dedicated fix; parking removes its hand-back trigger, and the `done` path nobody restarts after is unchanged.
- **Q15 — Update the hub's HANDOFF run loop?**
  Resolved, auto-pick: no; `manifest/HANDOFF.md` is updated only when the operator asks.
  Recorded under Scope Out as operator-owned (moved from Open risks in review round 1).
- **Q16 — New CONSTRAINTS.md invariant?**
  Resolved, auto-pick: yes, the Transient Stop Invariant (D12).
