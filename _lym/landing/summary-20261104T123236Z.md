# Reopen a beam from PR-Review, and name pull requests in beam history

Two changes to a beam's pull request lifecycle, from issues #46 and #44.

## `lym beam reopen <slug> [--close]`

A beam whose pawl run is past Beam-Await had no tested way to take new members. The new operator verb sends the landing back to Beam-Await in one step:

- It refuses unless the beam's pawl run is at Parent-Align, Beam-Describe or PR-Review. It also refuses a beam another machine owns, a beam with no pawl run, and everything `lym beam land`'s entry check refuses. Each refusal names its way forward.
- It writes a reopen request (`pawlengine.ReopenRequest`, `reopen-request.json` under the beam's `.lym/beam/`) that names the run by its seed's `started_at`, as a land request does.
- It removes the beam's ready record and commits the landing directory, so the dobby does not land the beam again. The commit runs even when no record was removed, so a rerun after a failed commit still commits the removal.

The run carries out the rest:

- **PR-Review** reads the request through a new `PullRequestDeps.Reopen` seam, on entry before the mode decision and at every wake of its wait. It stops the pr-reviewer. It then keeps an open recorded pull request open as a draft and posts a comment saying the landing was reopened. With `--close` it closes the pull request with a comment instead. A kept record loses the continue flag, the operator's ending, the mailed head and the session. A closed pull request loses its record, so the next landing opens a new one. The row then ends Stuck with `Reroute`.
- **The pawl recipe** gains its one reroute edge: `PR-Review on_reroute: Beam-Await`.
- **Beam-Await** removes the request on entry. For a request naming its own run, it starts the beam's dobby if it is down, since the landing stopped it. It also tells the orch that the beam is reopened and names `lym beam ready`, plus `lym beam start` if the dobby failed to start.
- On the next landing, Beam-Describe rewrites the description and PR-Review finds its recorded pull request open as a draft. PR-Review marks it ready for review before setting the title and body, so the pull request keeps its link, comments and history.

The request stays standing until Beam-Await removes it. If shed's reroute cap halts the run at PR-Review, a resume answers the request again instead of landing the beam.

`pullrequest.API` gains `SetDraft` and `Comment`, and `Status` gains `Draft`. GitHub's REST API cannot set draft state, so `SetDraft` reads the pull request's node id and runs the GraphQL `convertPullRequestToDraft` or `markPullRequestReadyForReview` mutation. An error listed in the GraphQL response counts as a failure.

## Pull request numbers in squash commits and the beam's description

- Finalize's merge on GitHub now writes the commit title as `<title> (#<number>)`, the form GitHub's own squash merge uses. A landed commit therefore names its pull request even after its board entry is pruned.
- Beam-Describe now lists the beam's commits under `## Changes`, oldest first. The list comes from the beam branch's own first-parent line, cut off where the parent's history begins. A commit whose subject ends in ` (#<number>)` appears as `#<number> <title>, merged <time>`. Every other commit, including a merge from the parent, appears as `<object name> <subject>, committed <time>`. Each entry has the first sentence of the commit body on an indented second line. GitHub links both the number and the object name in a pull request body. The seam that used to return commit subjects is now `BeamDescribeDeps.Commits` and returns `landingshed.BeamCommit`.

## Design decisions

- The verb hands its work to the run through a request file, the same way `lym beam land` uses a land request. While the PR-Review row is waiting, the run is running and holds its lock, so `lym loom goto` cannot move it. PR-Review answering the request itself also means only PR-Review, Finalize and pair teardown ever close a pull request, as `PATTERN-github-auth` requires.
- Drafting, commenting and closing happen in PR-Review, not in the verb. For a halted run, the verb's envelope names `lym loom resume` in the beam's worktree, after which PR-Review answers the request.
- Beam-Describe links pull requests and commits through GitHub's autolinking (`#266`, a 12-character object name), not explicit URLs, so it needs no origin read.
- A new `loomshed.NameBeamDescribe` constant names the Beam-Describe row, which the verb's list of admitted rows needs.

## Known limits

- Shed's reroute cap allows three reroutes per row since the row's last Done. A beam reopened a fourth time before PR-Review is Done again halts at PR-Review with the cap's own way forward.
- A reopen that fails between posting the comment and updating the record posts the comment again on retry.

## Tests

- `TestPRReview_Reopen`: drafting with a comment, a pull request that is already a draft, `--close`, a pull request closed on GitHub, a run with no pull request, a request seen at a wake, and a transient GitHub failure. A new `TestPRReview_Call` row covers a draft marked ready on re-entry.
- `TestBeamAwait`: a reopen that starts a stopped dobby, one with a live dobby, a failed dobby start, and a request for another run.
- `TestReopen` covers the verb's refusals, the accepted rows, `--close`, a halted run and a failed ready removal. `TestClient_Calls` covers `Comment`, `SetDraft` and `Status.Draft` against the scripted server.
- `TestFinalize_PullRequest` expects the ` (#7)` title. `TestBeamDescribe` covers the `## Changes` rendering. `TestBeamCommits_ListsTheBeamsOwnLine` (tier 1) runs `beamCommits` over real git: it checks that member and parent merges are listed and that commits a merge brought in are left out.
- The pawl recipe tests pin PR-Review's reroute edge.
