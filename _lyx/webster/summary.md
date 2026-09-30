# Post-merge verify gate for landing

All seven batches were already `done` when this session resumed, so it ran no batch loop.
The cards, in order:

- `01-verifyrun-runner`: shared in-process shell-command runner package.
- `02-webster-shared-runner`: webster's bisect verify now goes through the shared runner.
- `03-loomengine-verify-paths`: scratch accessors for the verify marker and output file.
- `04-landingshed-verify-gate`: told Deps fields and the post-merge verify gate.
- `05-publish-verify-gate`: gate runs between Publish's merge-in and its push.
- `06-finalize-verify-gate`: gate runs after every Finalize merge-in, before the parent-side merge.
- `07-loomcli-verify-wiring`: landing is told the plan's verify command and the two scratch paths.

The plan carries a `## verify:` section, so the integration fork ran the verify command once with no commit.
Its report says `status: OK`, `head_sha: 5f98c2cbb93c49c4e315d482bc167ea8ddece1d6`, `deviations: []`.
No deviations were reported and no friction note was written.
