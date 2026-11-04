# Land on a protected parent through a pull request merged at once

A repository whose `main` refuses a direct push, as NORCE Models' branch protection does, could take no landing: Finalize squashes onto the local parent and pushes it, and the push is refused.
beam.yaml's `pull_request` now takes a fourth value, `merge`, that lands such a parent through the remote with no review to wait for.

## Changes

- `internal/dobbyengine`: beam.yaml's `pull_request` accepts `merge` beside `on`, `off` and `required`.
  Set at the default layer (`lym config beam --layer default --set pull_request=merge` from the prime) it marks every branch of the hub; set in a worktree it marks that worktree's branch.
- `internal/landingshed`, `ResolvePullRequest`: a `merge` parent wants a pull request whatever the entry's field says, as `required` does, and its yes is marked `MergeAtOnce`.
  A missing origin fact halts, as under `required`.
- PR-Review: a `MergeAtOnce` yes logs and ends Done with no push, GitHub call, session or wait.
- Finalize: a `MergeAtOnce` yes with no pull request recorded no longer halts naming `lym loom goto PR-Review`.
  After the lock re-check and the task push, Finalize finds the open pull request from the task branch into the parent and sets its title and body, else opens one, and records it with the pushed head as gated and the review ended.
  It then merges the pull request through the existing GitHub merge path, and fetches and fast-forwards the parent, so the parent-side merge takes the already-landed path.
  Under `merge` the method is always squash, whatever landing.yaml's `squash` says, and a method refusal names the repository's settings in place of the `squash` key.
- Finalize's parent push: when GitHub refuses the push as a protected branch (its `protected branch` or `repository rule violations` rejection wording), the Stuck reason names `pull_request=merge` at the parent's worktree or the hub's default layer.
  It also names the hand landing of the work the local parent alone holds.
  That way forward replaces the push class's own way forward and the adopter-first repair note.
- Package docs of both packages and the beam.yaml template describe the new value.

## Design decisions

- The setting is a value of the existing `pull_request` key, not a new key: the resolver already decides per receiving branch from it, its layers already give a hub-wide default and a per-branch override, and `PATTERN-pull-request-mode` keeps the one resolver.
- Finalize opens the pull request itself, under the parent lock and after the owner checks and lock re-check, rather than PR-Review opening it: nothing is pushed or opened on GitHub from a machine that may not land the run, and the open sits next to the merge it exists for.
  The record Finalize writes carries the review as ended, so a re-entry after a crash merges the recorded pull request and reopens it if it was closed.
- The parent lock, the `finalize` gate over the merged tree and the board's done mark are unchanged; the GitHub merge reuses the existing path with its pending-refusal retry.
- The protected-branch way forward is read from git's rejection text, which landingshed classifies itself; fabric learns nothing of beam.yaml.

## Bounds

- A landing refused as a protected branch has already merged onto the local parent and marked its board entry done, so until the hand landing its work is on the local parent only.
  The way forward is: merge the task branch through a pull request on GitHub by hand, `git fetch origin` and `git reset --keep origin/<parent>` in the parent's worktree, then `lym loom resume`.
  The resumed Finalize finds the parent holding the work and ends the landing.
- The protected-branch wording is GitHub's English text; another host's or locale's rejection gets the plain push-failure way forward.
- Under `merge` lym merges with no approval; a parent whose protection demands an approval or a status check gets GitHub's refusal, and the existing halt names an approval, the check and `lym batten cancel`.

## Tests

- `TestLoadConfig`: `merge` loads; the refusal names the four values.
- `TestResolvePullRequest`: the `merge` mode with each entry field, the `MergeAtOnce` mark, the warning for an off field under `merge`, and a missing upstream under `merge` halting.
- `TestPRReview_Call`: a `merge` parent ends Done with no push and no GitHub call.
- `TestFinalize_PullRequest`: a `merge` parent with no record opens the pull request and squash-merges it with `squash` off; it finds and edits an open one on the task branch; a refused open halts and merges nothing; a method refusal names the repository's settings.
- Finalize's push table: a plain and a classified protected-branch refusal each end in the `merge` way forward with no repair note.
