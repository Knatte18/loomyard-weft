# Card 14 — driver-arbiter-branch

**Edit:**
- `contracts/stencils/shed/shed-template-driver.md`
- `internal/loomcli#driverPrompt`
- `internal/loomcli#TestDriverPrompt`
- `contracts/stencils#wordingClaims`
- `docs/overview.md`

**Create:**
- `internal/loomcli#driverArbiterCommand`

**Intent:**
Give the driver its arbiter branch (D6) through a new marker, keeping the stencil recipe-blind (D1).
Bound: the arbiter run is bounded by Go's eligibility refusal and the `arbiter_max_spawns` budget (cards 9, 10), and the re-launch moves the run only through the consumption of card 12, at most once per attempt (the applied mark), only backward, and never over a human's goto or pause.
A driver that runs the verb at a `producer`, `bootstrap` or reinvoke `interrupted` error without a fork escalation, which Go cannot check, skips the fork's diagnosis and the repair cap, bounded by the spawn budget, the applied-attempt refusal and the arbiter having no repair verbs (D2).

`driverArbiterCommand` returns `lyx loom arbiter <runID> --envelope <envelope_path> --report <report_path>`, the last two angle-bracket placeholders left for the driver to fill, as the stencil's `--before <producer>` is.
`driverPrompt` fills the new marker `arbiter_command` with it, beside `park_command`, so loom and darn runs both get it.
`TestDriverPrompt` asserts the marker is filled with the run id's command and no `{{` survives.

The stencil's banner lists `{{.arbiter_command}}`, and its body gains a `## Arbiter` section and the edits below, branching only on envelope fields:

- When: at a `blocked` stop, at an `awaiting` stop whose envelope carries a non-empty `parent_notice`, after the fork's action at an `error` or `interrupted` stop is an escalation, and at a transient stop that failed twice, run `{{.arbiter_command}}` as one background job from the session's own cwd, with the stop envelope's `envelope_path` and the current stop report path filled in.
  Never run it at any other stop, and never at an error stop whose fork named a repair verb or a re-launch.
  Wait for the job's exit as for the loop.
- `verdict: resume`: run the envelope's `relaunch` line at once as the loop's background job; this is a new entry in the closed re-launch list, allowed over a spent repair cap, and for this case it reverses the rule that a `blocked` stop is never repaired.
  The re-launch's stop is handled like any other stop.
- An error envelope from that re-launch carrying `arbiter_attempt`: take the hand-on path with the verdict path.
- `verdict: hand-on`, a refusal from the arbiter command other than the wake re-run's below, or the job's observed exit without an envelope: write the stop report with the arbiter's `reason`, `question` and `verdict_path` when present, or the refusal verbatim, then park and notify the parent exactly as before, the parent notice relayed verbatim at an `awaiting` stop.
- On a wake that finds the session unparked with no loop job and no arbiter job running, re-run the arbiter command only when the driver's own last action for the current stop was running that command and its job ended without an envelope the driver holds, as after a compaction or a driver revive mid-arbiter; never on the wake of the loop job's own exit.
  The re-run spawns nothing for a stop already arbitrated and returns that attempt's verdict.
  A refusal from that re-run falls back to the stop's existing handling under `## Stopping on a non-running state` or `## Error and interrupted stops`, never to a park quoting the refusal.

The existing sections change to match: `## Stopping on a non-running state` and `## Error and interrupted stops` point the `blocked`, escalated and twice-failed transient stops at `## Arbiter` before the hand-back, `## Re-launching` gains the `resume` entry, and `## Ways forward` keeps its sentence that `goto` is an operator and orchestrator verb the driver never runs, naming the re-launch's goto of a consumed `resume` attempt as the one move the driver's loop makes on an arbiter's verdict.

Add claims to the driver's `wordingClaims` row: `{{.arbiter_command}}`, `` `relaunch` ``, `` `arbiter_attempt` ``, `never run it` and `never on the wake of the loop job's own exit` stay named, the last pinning the wake re-run's narrowed condition.
The existing claims keep passing.
Update `docs/overview.md`'s driver paragraph under the shed module: at a stop it cannot settle, the driver runs `lyx loom arbiter`, which spawns a cold arbiter agent and returns a verdict the driver applies or hands on.

**ImpactSummary:** the driver prompt gains one marker and its stencil one branch; loom and darn runs get it alike.
