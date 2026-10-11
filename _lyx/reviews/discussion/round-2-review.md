---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md (D3 Bound; Open risks, second entry)
    summary: D3 calls a wrong-parent driverless seed stranded because no verb removes a seed, but the seed is an uncommitted file a plain rm clears, and the refusal step reaches names a looping way forward
---

### [MEDIUM] A wrong-parent driverless seed is not stranded, and the refusal on that path names a loop

**Location:** `_lyx/discussion/decision-record.md`, D3's Bound paragraph and the second `## Open risks` entry.

**Issue:** D3's Bound says a seed written with a wrong `<parent-branch>` makes every later `lyx loom step` refuse at its seed stage, "and no verb removes a seed", and Open risks records the pair as stranded.
The premise is false.
`lyx shed seed` (`writeSeed` in `internal/shedcli/seed.go`) only calls `shedrun.WriteSeed`, which writes `seed.json` under `_lyx/shed/<run>/` and commits nothing.
`seedAndCommitBootstrap` (`internal/loomcli/sharedbootstrap.go`) refuses at `bootstrapStageSeed`, before its step-3 commit, so the wrong seed stays an uncommitted file in the worktree.
A plain `rm <path>` of that file clears it, and `PATTERN-no-denied-recovery` names a plain `rm <path>` as a valid step that agents' settings do not deny.
Meanwhile, the refusal the agent hits is `ErrDisagreeingSeed` from `shedrun.WriteSeed`, whose way forward reads "keep the existing seed and drive it", the refusal-spec's `seed disagrees` row.
On this path, driving the existing seed is what `step` just refused, so the agent following the message loops.
The rubric's attack-surface rule wants D3 to state what the new route lets through and what bounds it.
D3 states the effect but bounds it with a recovery claim that does not hold, and leaves the plan writer no way forward to name.

**Fix:** In D3's Bound, replace "no verb removes a seed" with the real way forward: the wrong seed is uncommitted until a step succeeds, so deleting `seed.json` at the path the refusal names with a plain `rm`, then re-running `lyx shed seed` with the recorded parent, clears it.
Decide whether `step`'s seed-stage refusal names that way forward instead of the generic "keep the existing seed and drive it" clause.
If so, add it to D4's refusal changes, Scope and the refusal-spec rows, with an acceptance criterion.
If not, record the misleading clause, not a stranded pair, under Open risks.

## Focus verdicts

- Driverless seed route (D3, D4): consistent, not a defect.
  After `lyx shed seed self --recipe loom --driver go --param parent=<parent-branch>`, `resolveRunID` finds the seed, so `step` passes arm.
  `loomPreStep` then runs `seedAndCommitBootstrap`, which writes the status file.
  `run` before any `step` hits `loomPreRun`'s status-file refusal, and D4 adds `lyx loom step` to that refusal's remedy.
  D3, D4, Scope ("`run`'s status-file refusal's way forward" and its new spec row) and the acceptance criteria all give the same order: seed, then `step`, then `step` or `run`.
- D1 turn-start rule and long gaps: not a defect.
  The record keeps a gap under 30 minutes that ends at an assistant, `tool_result` or interrupt-marker line as working time, gives up any gap over the cap, and lists the first case under Open risks.
  Its Rationale cites the observed transcript stalls.
- D1 and compaction: not a defect.
  I checked four real transcripts under `~/.claude/projects/` that hold `isCompactSummary`.
  Sorted by timestamp, each runs: a meta `<local-command-caveat>` line, the `/compact` command line at the same instant, the summary line about one to five minutes later, then the `<local-command-stdout>` line.
  The `/compact` line does not end the turn under `endsTurn`, and the summary line is excluded from turn-starts.
  So the interval between them stays working time, as D1 and its acceptance criterion say.
  The Bound's mid-turn claim is not contradicted, because a `/compact` command line is a real turn start and not a mid-turn line.
- D3, D4 and the typed parent branch: see F1.
- D2 and empty transcripts: not a defect.
  D2 decides the case: with a bound, a main-session transcript with no timestamped user or assistant line is left out and adds nothing to the session count.
  Without a bound, it is counted as today.
  The acceptance criterion matches.
