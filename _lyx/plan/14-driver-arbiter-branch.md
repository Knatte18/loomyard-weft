# Card 14 — driver-arbiter-branch

**Edit:**
- `contracts/stencils/shed/shed-template-driver.md`
- `internal/loomcli#driverPrompt`
- `internal/loomcli#TestDriverPrompt`
- `contracts/stencils#wordingClaims`
- `docs/overview.md`

**Create:**
- `plan:internal/loomcli#driverArbiterCommand` -> `func driverArbiterCommand(runID string) string`

**Intent:**
Give the driver its arbiter branch (D6) through a new marker, keeping the stencil recipe-blind (D1).

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
- `verdict: hand-on`, a refusal from the arbiter command, or the job ending without an envelope: write the stop report with the arbiter's `reason`, `question` and `verdict_path` when present, or the refusal verbatim, then park and notify the parent exactly as before, the parent notice relayed verbatim at an `awaiting` stop.
- On a wake that finds the session unparked, with no loop job and no arbiter job running, re-run the arbiter command for the last stop; it spawns nothing for a stop it already arbitrated and returns that attempt's verdict.

The existing sections change to match: `## Stopping on a non-running state` and `## Error and interrupted stops` point the `blocked`, escalated and twice-failed transient stops at `## Arbiter` before the hand-back, `## Re-launching` gains the `resume` entry, and `## Ways forward` keeps its sentence that `goto` is an operator and orchestrator verb the driver never runs, naming the re-launch's goto of a consumed `resume` attempt as the one move the driver's loop makes on an arbiter's verdict.

Add claims to the driver's `wordingClaims` row: `{{.arbiter_command}}`, `` `relaunch` ``, `` `arbiter_attempt` `` and `never run it` stay named.
The existing claims keep passing.
Update `docs/overview.md`'s driver paragraph under the shed module: at a stop it cannot settle, the driver runs `lyx loom arbiter`, which spawns a cold arbiter agent and returns a verdict the driver applies or hands on.

**ImpactSummary:** the driver prompt gains one marker and its stencil one branch; loom and darn runs get it alike.
