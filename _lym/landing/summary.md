# Rewrite a beam's PR description after each member landing

While a beam's pull request is recorded, the beam's dobby now keeps the pull request's description current on its own.
Until now the orch wrote `summary.md` by hand, and only a landed line asked it to update the description.
That line went out only when the member's `land_notice` was on, so with notices off the description fell behind.

## What changes

- **Dobby's description rewrite** (`internal/dobbyengine/describe.go`).
  Each landing a beam's dobby counts goes on a new `landed_since_describe` list in its state record.
  When the beam's landing record holds a pull request, dobby starts a describer agent.
  It does not start one while another describer agent is running.
  The agent reads the recorded description, the board entry, the beam's diff and its commit list, where each member pull request appears by number.
  It rewrites the description whole into a draft under `.lym/beam/describe/<UTC timestamp>.md`.
  Once the draft validates, dobby ends the agent and records the draft.
  Landings counted while no pull request is recorded are dropped, and the prime's dobby rewrites nothing.
- **Recording from a draft** (`internal/beamcli/describe.go`, `internal/landingshed/beamdescribe.go`).
  `rewriteDescription` applies the same checks as `lym beam describe`: an open beam of this machine, a pawl run at Beam-Await, and no ready record.
  `landingshed.ReplaceBeamDescription` then validates the draft, writes it over `summary.md` and commits it.
  With a pull request recorded, the `pr update` sequence follows, as for the verb.
  A draft with a finding leaves `summary.md` as it was, and a failed commit puts back the previous file.
- **Failure handling.**
  A refused record, or an agent with no valid draft by its timeout, leaves the recorded description in place.
  Dobby then posts one line to the nearest live orch saying why.
  The next member landing tries the rewrite again.
  The orch and the operator can still edit and record by hand with `lym beam describe`.
- **Landing notice.**
  The landed line no longer carries the "add it to the change description, then run `lym beam describe`" follow-up.
  `land_notice` now decides only whether the orch hears of a landing.

## Design decisions

- The rewrite is a dobby step modelled on the board tidy, not a row of the member's landing.
  Dobby already counts the landings of the beam's members, and it owns the beam's worktree, where the description and its commit live.
- The describer agent shares the Describe row's settings: landing.yaml's `describe` model, `describe_timeout_min`, and the `describer` config role with its skills.
  Its prompt is a new stencil, `beam-template-describe`, with role `beam-describe`.
  Its shuttle run uses the Describe row's description gate, re-prompting up to three times.
  Dobby records the draft only after the draft validates.
- The agent writes a separate draft, never `summary.md`, so a failed rewrite cannot damage the recorded description.
- A landing counted while a board-tidy agent is running waits until the tidy settles before its rewrite starts.
  This follows from the landing count, which counts nothing while a tidy is in flight.
- Dobby's shuttle adapter is renamed `runnerAgentShuttle`, and its interface `AgentShuttle`, because the tidy and the describer agents now share it.
  The adapter takes an optional gate.

## To check by hand

- No real beam has run this yet.
  Land a member into a beam whose pull request is open, and confirm that the agent starts.
  Confirm that `summary.md` is recommitted and that the pull request's body changes.
- Confirm that a beam flagged ready refuses the rewrite and that its orch receives the "was not recorded" line.

## Merge danger

**Door:** two-way
**Blast radius:** dobby
