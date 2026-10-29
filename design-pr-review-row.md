# PR-Review opens a pull request and starts an interactive agent to discuss it

`pr-review-row` · Task · feature, loom, shed, batten, shuttle, config · awaiting · Discussion-Bouncer

## Goal

Build the `PR-Review` row, so a task can land through a pull request that the operator discusses with an interactive agent before it is merged (operator, 2026-10-29).

## Seen

`PR-Review` is a stub today: `internal/landingshed/doc.go` calls it the human pull-request review that is not built yet, and `Finalize` opens, looks up and closes no pull request, so a landing makes no GitHub call.
A hub on Models needs a pull request to review before a run lands.

## Design

- The row opens a pull request for the task branch against its parent and then starts an interactive agent session the operator can talk to, to discuss the pull request.
  The session has its own config key for the engine, model and effort it runs on, in the same `<alias>[<effort>]` form `lym shuttle open --model` takes.
- The row sits where the stub already sits in the recipe, before Finalize.
- Whether a task goes through a pull request is decided by the parent branch it merges into, never by a profile; profiles name only review rounds and carry no value for this.
  Each branch, `main` and every beam, has one of three values, set in the config layer of the receiving branch:
  - `on`: a pull request is the default, and a task may override it to off through a field of its board entry.
  - `off`: no pull request by default, and a task may require one through the same field.
  - `required`: every merge into the branch needs a pull request and no task can turn it off; lym's own lock, which binds only runs that go through lym.
  The default for a repository that sets nothing is `on`, also for `main`.
- `PR-Review` reads the parent's value first, then the task's override, and checks the override is allowed under that value.
  A landing of a beam into its own parent follows that parent's value, so a beam set to `off` for its tasks does not let the beam reach `main` without a pull request when `main` is `on` for beams or `required`.
- The run follows the pull request's state, whatever happens to it: the operator approves it on GitHub, someone else approves it, or someone has already squashed it; all three end in the same landing.

## Landing with a pull request up

Finalize today takes the parent lock, syncs the task branch with the parent, squashes it onto the parent, marks the board task done and pushes the parent, all in one go.
With a pull request up:

- The parent lock is never held while the run waits for the discussion or the approval.
- The parent may move during the wait, so the pull request can conflict and must be brought level again, which Finalize's merge fixer already does under the lock.
- Finalize merges the pull request instead of squashing onto the parent itself, or notes that it is already merged, and only then marks the board task done, pulls the prime and lets the teardown run.

## Open for Discussion

- Whether the interactive agent may change the branch during the discussion, and if so how the change is gated and committed.
- How the waiting run is shown to dobby and the orch, so it reads as waiting and not as stuck.
- Which credentials open and read the pull request: the hub's own repository may belong to another GitHub account than the session's, as Models does.
- What happens to the pull request when the run is cancelled or the operator rejects it.
- What the interactive agent is given: the diff, the plan and the decision record, which it reads itself, never as pre-read text in its prompt.
- Whether lym should also read GitHub's branch protection of the parent as the source of `required`; left outside the first build.
