# PR-Review opens a pull request and starts an interactive agent to discuss it

`pr-review-row` · Note · feature, loom, shed, shuttle, config

## Seen

`PR-Review` is a stub today: `internal/landingshed/doc.go` calls it the human pull-request review that is not built yet, and `Finalize` opens, looks up and closes no pull request, so a landing makes no GitHub call.
The operator wants it built first, since a hub on Models needs a pull request to review before a run lands (operator, 2026-10-29).

## Idea

The PR-Review row opens a pull request for the task branch and then starts an interactive agent session the operator can talk to, to discuss the pull request before it is merged.
The session has its own config key for the engine, model and effort it runs on, in the same `<alias>[<effort>]` form `lym shuttle open --model` takes.
The row sits where the stub already sits in the recipe, before Finalize, and its flag keeps the stub's meaning of off by default.

## Landing with a pull request up

Finalize today takes the parent lock, syncs the task branch with the parent, squashes it onto the parent, marks the board task done and pushes the parent, all in one go.
With a pull request up that cannot stay, since the merge is the pull request's and the run waits on a person for hours or days:

- The parent lock is never held while the run waits for the discussion.
- The parent may move during the wait, so the pull request can conflict and must be brought level again, which Finalize's merge fixer already does under the lock.
- Finalize then merges the pull request instead of squashing onto the parent itself, and only after that marks the board task done, pulls the prime and lets the teardown run.
- A pull request merged by hand on GitHub must reach the same end.

## Open

- Who merges: lym on the operator's word in the discussion session, the operator on GitHub, or either.
- Whether the interactive agent may change the branch during the discussion, and if so how the change is gated and committed.
- How the waiting run is shown to dobby and the orch, so it reads as waiting and not as stuck.
- Which credentials open and merge the pull request: the hub's own repository may belong to another GitHub account than the session's, as Models does.
- What happens to the pull request when the run is cancelled or the operator rejects it.
- What the interactive agent is given: the diff, the plan and the decision record, which it reads itself, never as pre-read text in its prompt.
