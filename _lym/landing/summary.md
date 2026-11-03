# Add an assistant reviewer seat to the Burler review round

A review round can now run an assistant reviewer beside its chair reviewer, on the engine its model choice names, Codex included.
The assistant works alone and writes one report.
The chair waits for the assistant's end marker, then consolidates the report as one more input.
The assistant never gates the round: a missing, failed or stopped assistant only means the chair goes on without it.

The feature is off by default.
Each review segment's `*_assist_reviewer_model` key is empty until an operator sets it.

## What changes

- `seatengine` gains an assistant seat kind.
  - `lym` writes the seat's end marker once its run ends, whatever the ending: `done`, `gate failed`, `stopped`, `not started`, `not found` or a wait error.
  - A fresh run clears a previous run's marker and report.
  - A resume reports an assistant it finds neither live nor ended as `not found`.
  - The chair's seat directive lists its assistants with their reports and end markers.
    The chair waits with `lym shuttle await <end marker>`.
  - The assistant's own directive keeps it to its output files and has it ignore `[creel]` lines.
- `burlerengine` builds the assistant for the review half.
  - It derives the assistant's report and end marker from the review path and the attempt token, under the anchor's `.lym`, so a retry never reuses the first attempt's files.
  - It composes the assistant's instruction files for the assistant's engine.
  - It adds a consolidation part to the chair's stencil through the optional `assist_rules` marker.
  - Once the chair has ended, it copies a `done` assistant's report beside the review as `<review stem>.assist-<seat>.md`.
- Engine-specific stencil blocks under `contracts/stencils/engine/{claude,codex}`: the edit directive, the reading directions and the subagent fan.
  - A Codex assistant fans its lens subagents with `spawn_agent` and `wait`, and a Claude one with the Agent tool.
  - Both finish the fan inside the assistant's one turn.
  - `internal/editdirective` renders the engine's own edit directive for an assistant seat.
- Config:
  - `loom.yaml` gains `discussion_assist_reviewer_model`, `code_assist_reviewer_model`, the matching `*_assist_reviewer_fan` keys and `assist_reviewer_timeout_min`.
  - `tapestry.yaml` gains `plan_assist_reviewer_model` and `plan_assist_reviewer_fan`.
  - Model and fan lists take one entry per round, the last entry serving later rounds, and an empty model entry runs no assistant that round.
  - Entries are parsed at load, and fan names are checked against `lenses.yaml`.
  - `docs/overview.md` is updated.
- `shuttleengine.Config.StartAllowance` returns the longest a start with its retries may take.
- `shedadapters` carries each segment's assistant settings into the Burler review row, and records the assistant's usage with the round's.
- `loomcli` computes each segment's assistant bound and hands it to the recipe's segment settings.

## Design decisions

- **Timeout cap.**
  `loomengine.CapAssistTimeout` caps the assistant so its start allowance, its run and one await cap end before the chair's own timeout.
  - The chair's timeout is `review_timeout_min`, or shuttle's `run_timeout_min` when that key is 0.
  - A cap that is hit logs one Warn.
  - When no room is left, the assistant does not start and its end marker names both keys.
  - For `assist_reviewer_timeout_min`, 0 means the whole room left, not "defer to shuttle" as for the other timeout keys.
    The config load refusal message states each key's own zero meaning.
- **Await cap per segment.**
  The chair's await cap is the keep-warm cap for the chair's role.
  Each review segment's recipe rows set `role_prefix` to their perch's name, so `segmentAssistBounds` resolves the cap under that prefixed role.
  A chair name that cannot be formed leaves the default await cap.
- **Assistant report path.**
  It is never tracked and sits under `.lym`, so it needs no write access to a checkout other than the worktree's own.
- **Placeholder files on a failed compose.**
  When composing the assistant's instructions fails (a missing engine block, an unknown fan), the seat gets placeholder paths so table validation passes, and its end marker reads `not started` with the reason.
  Nothing writes or reads those paths.
- **An assistant's report is advice.**
  The chair's directive says the report carries no authority over it.
  Only a `done` ending allows the chair to read the report.

## Check by hand

- `loomcli` wiring, `segmentAssistBounds`: a review finding on the keep-warm await cap for the assistant bound (`keep-warm-cap-wrong-chair-role`, blocking) is not recorded as closed.
  A later change resolves the cap under the chair's prefixed role.
  Confirm that the role `burlerengine.ReviewChairRole` forms from each perch name is the role `shuttleengines.KeepWarm` sees for the chair's actual strand, for the Discussion, Plan and Code segments.
- Run a real Burler round with a Codex assistant and one with a Claude assistant (`go test` for `internal/burlerengine`'s `smoke_assist_test.go` is a live-engine smoke test and was not run here), and check that:
  - the chair waits on the end marker;
  - the report is copied beside the review;
  - a fan finishes inside the assistant's turn.
- Check that a `not started` assistant, with a bad model entry or an unknown fan, leaves the round going without it.
- Run `lym config reconcile` on an existing hub and check that the new keys appear with their defaults.
