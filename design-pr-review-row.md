# PR-Review opens a pull request and starts an interactive agent to discuss it

`pr-review-row` · Note · feature, loom, shed, shuttle, config

## Seen

`PR-Review` is a stub today: `internal/landingshed/doc.go` calls it the human pull-request review that is not built yet, and `Finalize` opens, looks up and closes no pull request, so a landing makes no GitHub call.
The operator wants it built first, since a hub on Models needs a pull request to review before a run lands (operator, 2026-10-29).

## Idea

The PR-Review row opens a pull request for the task branch and then starts an interactive agent session the operator can talk to, to discuss the pull request before it is merged.
The session has its own config key for the engine, model and effort it runs on, in the same `<alias>[<effort>]` form `lym shuttle open --model` takes.
The row sits where the stub already sits in the recipe, before Finalize, and its flag keeps the stub's meaning of off by default.

## Open

- How the run goes on after the discussion: the operator's word, the pull request merged on GitHub, or both, and what Finalize does with a pull request that is already merged.
- Which credentials open the pull request: the hub's own repository may belong to another GitHub account than the session's, as Models does.
- What the interactive agent is given: the diff, the plan and the decision record, which it reads itself, never as pre-read text in its prompt.
- What happens to the pull request when the run is cancelled or the operator rejects it.
