# Case brief: origin-reconcile stuck at Worktree-Teardown

## The question

Why does the push of the archive tag `archive/origin-reconcile/3b91b9712fa0` to `https://github.com/Knatte18/loomyard-weft` get rejected as `remote rejected ... (failed)`, and is a plain re-launch of `lyx batten run origin-reconcile` enough to get past it?

## The facts

- The stop is `Worktree-Teardown`, state `blocked`, interrupt policy `handback`, history length 24 (`_lyx/shed/origin-reconcile/drive-reports/drive-envelope-20261021-152304-5ff2.json`).
- The step trace shows the archive push failing at 15:23:03.976, after the pair's logs were archived (`.lyx/shed/origin-reconcile/steps/27d50cf4600b5c2d.trace.log`, the `pairteardown: archived the pair's logs` line, 256 files).
- The git output is `exit 1`, then `! [remote rejected] archive/origin-reconcile/3b91b9712fa0 -> archive/origin-reconcile/3b91b9712fa0 (failed)`, with no reason text after `(failed)` (same trace file, `battenshed: producer stuck` line).
- The remote branch `refs/heads/origin-reconcile-weft` is at `3b91b9712fa0a9a93253383a3f580c7b3dcd4462`, which is the commit the teardown recorded (`fabric: mutation ... commit_created ... 3b91b9712fa0a9a93253383a3f580c7b3dcd4462`). So the branch push went through.
- At the time of this read, `git ls-remote origin 'refs/tags/archive/origin-reconcile/*'` returned no tag, so the tag is absent on the remote.
- The local tag `archive/origin-reconcile/3b91b9712fa0` exists in `loomyard-weft` and points at the same commit.
- Other `archive/*` tags are present on the remote (for example `archive/batten-notice-trim/e164998b0a96`), so tag creation is not blocked for every tag.
- A detached fabric push ran at 15:23:03.968 with `--teardown-slug origin-reconcile` (pid 1999746), in the same second as the failing push. The step's own `fabricengine: spawned detached push` line shows it. Its outcome is not in the trace.
- The stderr log of the step has no content beyond the trace copy.
- Envelope `loop.detail` and the stuck message both say to fix the named failure and resume with `lyx batten run origin-reconcile`.
- The run's `status.json` `error` field repeats the same text. An earlier `automatic Publish verify retry` entry shows an earlier resume at `Webster-Burler-Review`, not at this row.

## What I ruled out

- Unreachable remote: ruled out. `git ls-remote origin` answered, with refs and tags listed.
- Auth for tag pushes in general: not shown to fail. Other tags exist on the remote, but I did not check whether they were pushed by the same account.
- A tag already present on the remote at a different object: ruled out. The remote has no such tag.
- Local tag mismatch: ruled out. The local tag points at the teardown commit.
- Commit missing on the remote: ruled out. The branch is on the remote at that commit.

## Hypothesis (not verified)

- Hypothesis: the rejection is a server-side refusal of the tag ref, either a repository ruleset or tag protection on `archive/*`, or a race with the detached `fabric push` at pid 1999746 that also touches the same archive path.
- The race reading would make a plain re-launch succeed, since the tag is now absent and the branch is already in place.
- The ruleset reading would make a re-launch fail the same way.
- A judge with read access to the remote's rulesets, or the ability to run `git push --dry-run` against the tag ref without changing anything, can separate these two readings.
- I could not separate them from the files and read-only commands I had.
