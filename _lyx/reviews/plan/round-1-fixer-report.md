# Fixer report — plan, round 1

Every finding in `round-1-review.md` is fixed in the plan cards.
The overview, including its `approved:` key, is untouched.

## Changed

- **F1** (`03-move-begun-card-hashes.md`): the Bound now says no site restamps after a failed check.
  It states the residual: an edit landing between the check and the restamp in one call is adopted with the rewrite, begun-card hashes included.
  It names that window per site.
- **F2** (`09-integration-fix-attempt.md`): the stuck reason and the new refusal-spec row carry the full way forward, ending with re-running `lyx webster run` (re-step the Webster row).
  A reaching test asserts the stuck reason of an unfixed and of a spent attempt names the pre-fix head, each fix commit, `git reset --hard`, the verify-and-commit step and the re-run.
- **F3** (`09-integration-fix-attempt.md`): a new step 0 checks `FixStarter` before anything is recorded.
  A render error joins a start error as a failed attempt recorded in phase 3.
  The free-standing nil-`FixStarter` paragraph is gone, and a test asserts the wiring error leaves `IntegrationFix` unrecorded.
- **F4** (`07-fix-commit-check.md`, `09-integration-fix-attempt.md`): card 7's path rule covers the `_lyx` entry itself.
  A new Bound paragraph states that commits cannot reach a plan directory or `_lyx` outside the repository, and what bounds that case.
  Card 9 captures the plan fingerprint at step 1 and refuses the attempt at step 5 when it changed; tests cover a root `_lyx` entry and an uncommitted plan edit.
- **F5** (`05-integration-fix-prompt.md`): the prompt forbids writing under the plan directory and committing under `_lyx`.
  It names the report and the friction note as the only `_lyx` writes, and says to leave nothing uncommitted.
  The render test asserts the report path is the stated exception.
- **F6** (`09-integration-fix-attempt.md`): an attempt found in flight (empty `Result`) when the stage runs ends as `failed`, whatever the integration report's status.
  The stage records its commits and the summary section, sets `Result` and escalates.
  The refusal row's trigger and the Bound name it, and a test covers an in-flight record over an `OK` report.
- **F7** (`07-fix-commit-check.md`, `09-integration-fix-attempt.md`): `checkFixCommits` refuses a dirty worktree through the existing `dirty` helper, and both cards' tests cover it.
- **F8** (`01-dispatch-forthcoming-targets.md`): forthcoming cards are selected from the reloaded plan, or from `plan` when nothing was rewritten, never from `current`.

## Deferred

None.
