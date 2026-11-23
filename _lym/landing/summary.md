# Rewrite a beam's pull request description after each member landing

While a beam's pull request is recorded, the beam's dobby now spawns a describer agent after each member landing.
The agent reads the recorded description, the beam's board entry, its commits and its diff, and rewrites the description whole as the beam now stands.
Dobby validates the draft and records it as `lym beam describe` records an orch's text, which also updates the pull request.

Until now the landed line asked the orch to edit `summary.md` and run `lym beam describe <slug>`.
With `land_notice` off the orch never got that line, so the pull request's description fell behind.
No orch is asked any more: the landed line drops its follow-up.

## What changes

- `internal/dobbyengine/describe.go` adds the describe step.
  A landing counted while a pull request is recorded joins the landings to describe.
  One agent covers every landing counted before it starts, and a landing counted while it runs gets a rewrite of its own.
  A landing counted while no pull request is recorded is dropped.
- A rewrite that fails leaves the recorded description standing, and the next landing retries it.
  A spawn that fails, a draft that never passes by its deadline and a record that fails each post one line to the nearest live orch.
  A failure of this step is logged and holds neither the tidy step nor the landing step.
- A line the step owes is stored in the state record before it is posted, so a failed post is retried without recording again.
- The agent is stopped when its beam's worktree is gone, as the board-tidy agent is.
- `contracts/stencils/beam/beam-template-describe.md` is the agent's prompt.
  It points at the final-summary spec for the format and asks for the end result, every member pull request by number and a Merge danger section called for the beam's whole landing.
- `internal/dobbyengine/state.go` adds `landed_since_describe` and `describe` to dobby's state record, both `omitempty`.
- `internal/landingshed/beamdescribe.go` adds `ReplaceBeamDescription`, which validates a draft and commits it over the description, putting the old text back when the commit fails.
  It also exports `RenderBeamChanges`, which the prompt's commit list shares with `lym beam describe --draft`.
- `internal/beamcli/describe.go` splits `describe` into the shared refusals, `judgeDescribable`, and the shared record sequence, `recordDescription`.
  `RecordDescription` in `landDeps` now takes the draft path to record from.
  A failure after the description is recorded is typed as such, so dobby can tell a missed pull request update from an unrecorded description.
- `internal/dobbyengine/landnotice.go` loses the follow-up and its read of the landing record.
- `internal/beamcli/dobbydeps.go` wires the seams: the describer role's model, timeout and findings check.
  `docs/overview.md`, `contracts/orchrole/role-beams.md` and the `dobbyengine` and `beamcli` package docs follow.

## Design decisions

- The rewrite runs in dobby after the landing, not as a row of the member's landing run.
  That keeps it independent of `land_notice` and of the member run's lifetime.
- The describer reuses the `describer` agent role from `agents.yaml`, and its own stencil states the beam-specific content rules.
- The description is rewritten whole, never appended to, as the operator decided on 2026-11-22.
- The Merge danger blast radius stays the one word the final-summary spec allows.

## To check by hand

- The describer agent has not run against a real beam and pull request here, so check one landing under a recorded pull request end to end:
  the description is rewritten and the pull request shows it.
- `internal/dobbyengine/describe.go`: a describe error must not block the tidy step or the landing step.
  Check the loop in `loop.go` against the pinned behaviour in `describe_test.go` and `loop_worktreegone_test.go`.
- `internal/dobbyengine/describe.go`: a failed pull request update after the record must name `lym beam pr update <slug>`, not "unrecorded".
- `internal/dobbyengine/describe.go`, `settleDescribe`: an unreadable draft must not keep the attempt in flight past its deadline.
- `internal/dobbyengine/loop.go`: a beam dobby whose worktree is gone must stop a describer agent in flight.
- `internal/beamcli/dobbydeps.go`: the tick wiring of the describe step is pinned by `dobbydeps_test.go` and the integration test; check that the seams it sets are the ones the loop reads.
- `contracts/stencils/beam/beam-template-describe.md`: it must point at the final-summary spec and not restate its format.

## Merge danger

**Door:** one-way
**Blast radius:** beam
