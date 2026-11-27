# PR-Review keeps an agent session in the task's window in review mode

Under `beam.pull_request: review` the PR-Review row opened the pull request and then waited with no agent session in the task's window.
Nothing on screen showed that the run waited on the operator, and no agent was there to take the operator's feedback on the pull request.
`lym loom pr-continue` also refused every agent session, the orch's included, so the review could only be ended from a shell outside every session, by an approval or a merge on GitHub, or by editing the record by hand.

The row now keeps one interactive session in the task's window again, as it did before the session was removed.
The operator works the pull request with that session's agent, which answers comments, fixes findings and ends the review only on the operator's word.
`trail` and `off` are unchanged: `trail` still opens and squash-merges the pull request at once in Finalize, and `off` opens none.

## What changes

### The PR-Review session

- Once the pull request is recorded, the row starts the session, reuses a live one, or resumes a recorded one by its session id.
  It records the session's strand and session id in `pull-request.json`, so a re-entry after an interrupt attaches to the same session.
- A recorded pull request that GitHub reports closed gets a session too, so a continue flag can still end it.
  A merged one and an already ended review get none.
- The session is stopped on every verdict, Stuck or error of the row, and left running when the row's context is cancelled.
  It is stopped before each read that may end the review, so the tree cannot change between the read and the verdict, and resumed when the review goes on.
- A pause stops the session and the end of the pause resumes it.
- A session that ends while the review is open halts the row; `lym loom resume` resumes it.
- A commit made in the task worktree while the review waits is still gated and pushed.
  A gate failure is now mailed to the session once per head and recorded as `mailed_head`, instead of halting the row.
  A runner crash or memory pressure stop still halts.
- The session's prompt, `contracts/stencils/landing/landing-template-pr-review.md`, has the agent read the change, the description and the regression check.
  It may comment on the pull request and change its title and body through the change description.
  It runs under the publish shell guard, so it never pushes, approves, merges, closes or reopens the pull request.
  The parent-review asks and the send-back of findings to Code-Review that the old session had are not restored.

### Ending the review

- `lym loom pr-continue` admits the operator's shell, the run's `pr-reviewer` session and an orch session.
  It refuses every other agent session, with a way forward that names the session in the task's window.
- It also clears `mailed_head`, so the row gates that head again after a failure from outside the change.
- Its help audience is the role audience again, since agents run it.

### Config

- The agents role `pr-reviewer` is listed again, with the prose, conversation, code-quality and testing skills.
- `landing.pr_reviewer` is a live key again, `opus[high]` by default, open to the local layer and checked as a model-spec at load.
- The reinterpretation `agents-pr-reviewer-role-retired` is dropped, so a hub's own `pr-reviewer` entries stay.

### Docs and rules

- `PATTERN-landing` now says a pull request's review ends only on the operator's word, whatever the run's autonomy.
  The operator gives that word directly, or the PR-Review session or an orch acts on it through `pr-continue`.
- The orch role files, `docs/overview.md`, the `landingshed` and `loomcli` package docs, the beam template's `pull_request` comment and the recipe comments follow.

## Design decisions

- **Old names, not new ones.** The role and the config key come back as `pr-reviewer` and `landing.pr_reviewer`, the names the removed session used.
  The reinterpretation that stripped the role from a hub's `agents.yaml` is dropped rather than kept.
  A hub that already ran it has lost only entries the template now supplies again.
- **The orch may end a review on the operator's word.** This gives the orch a sanctioned way to act on an operator's instruction instead of editing `pull-request.json` by hand.
  The check reads the strand name in the session's environment, so it guards against a mistaken call and is no barrier.
- **The session ends the review; Finalize merges.** The session runs `pr-continue` and never merges, and Finalize merges the pull request on GitHub under its own gate.
  GitHub refuses an author's approval of their own pull request, so `pr-continue` is what ends the review.
- **Gate failures mail, not halt.** With an agent in the window, a halt would stop the session the operator is working with.
  The failure is mailed once per head instead, and the operator's agent fixes it or retries with `pr-continue`.
- **Interactive whatever the autonomy.** `review` is the operator's review, so the session is interactive even when the run's autonomy is on.

## Merge danger

**Door:** one-way
**Blast radius:** landing
