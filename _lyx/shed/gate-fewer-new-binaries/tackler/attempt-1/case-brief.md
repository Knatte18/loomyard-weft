## Question

Why does the push of the archive tag `archive/gate-fewer-new-binaries/88d95b1f01f5` to `origin` (`https://github.com/Knatte18/loomyard-weft`) fail with `[remote rejected]`, and what has to change on the remote or in the run so that `Worktree-Teardown` completes?

## Facts

- The stop envelope `_lyx/shed/gate-fewer-new-binaries/drive-reports/drive-envelope-20261020-124203-9fe4.json` reports `state: blocked` at `Worktree-Teardown`, step 8 of 8, with the reason "worktree removal failed ... archiving the pair's records before teardown failed".
- The step trace `.lyx/shed/gate-fewer-new-binaries/steps/ed6b8d23ad114e2f.trace.log` at 12:42:03.477 logs `battencli: teardown worktree` with `archive_tag=""` and `archive_skipped_reason=""`, then the `battenshed: producer stuck` warning.
- The git output in that trace reads `! [remote rejected] archive/gate-fewer-new-binaries/88d95b1f01f5 -> archive/gate-fewer-new-binaries/88d95b1f01f5 (failed)`, with no reason text from the server.
- The trace at 12:41:55.954 records the commit `88d95b1f01f5358b4c08087179d39827a4113015` created on `gate-fewer-new-binaries-weft`. The 12:42:03.667 line records a second commit on `loomyard-weft`, `38793a629221fa7e2096f2fe06ecefed59098921`, whose push was spawned at 12:42:03.677. Whether that push succeeded is not in the trace.
- `git ls-remote --tags origin 'archive/gate-fewer-new-binaries*'` run in `loomyard-weft` returns nothing, so the tag is absent on the remote.
- `git tag -l 'archive/gate-fewer-new-binaries*'` in the same repo lists `archive/gate-fewer-new-binaries/88d95b1f01f5`, so the tag exists locally.
- The earlier trace lines for this run, `.lyx/logs/trace-20261020T104155Z-ed6b8d23ad114e2f-3986542.log`, contain no other archive push attempt.

## Ruled out

- A duplicate tag already on the remote: `ls-remote` shows none, and the rejection text does not say "already exists".
- Tag creation failing locally: the tag exists locally.
- The remote being unreachable: the server answered and rejected the ref, which the producer's own hint ("most often an unreachable remote") does not match.
- A re-launch alone settling it: this stop is the first stop on the teardown, but the rejection is a server-side decision and nothing in the logs shows it changing.

## Hypothesis (not verified)

The GitHub remote rejects pushes of `archive/*` tags for this account or repository, through a repository ruleset, a tag protection rule, or missing write permission on tags. The producer's hint is the generic one and does not match this case. A judge with access to the GitHub settings for `Knatte18/loomyard-weft` can confirm or refute this. Until the remote accepts the tag, a `resume` will fail at the same step.
