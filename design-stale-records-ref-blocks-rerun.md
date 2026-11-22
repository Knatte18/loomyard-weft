# A stale records tracking ref blocks a cancelled slug from running again

`stale-records-ref-blocks-rerun` · Task · bug, fabric, batten, dobby · running · Code-Burler-Fix

## What was seen (2026-11-22)

`harden-fabricengine` was cancelled with `lym batten cancel --remove` on 2026-11-21 and released again after the hardener rebuild landed.
Dobby refused to start it three times: `the code tip 3a9f8d84 the run's last transition names is not on origin yet`, though 3a9f8d84 is on origin/main.

Cause: the records repository still held the remote-tracking ref `origin/harden-fabricengine-weft` of the cancelled run, while origin had deleted the branch.
`fabricengine.OriginCodeTip` reads that ref, finds the old run's last transition and judges its code tip missing.

Workaround: `git -C <hub>/loomyard-weft fetch --prune origin`, which also dropped two other stale refs.

## Expected

- `lym batten cancel --remove` and the remote cleanup drop the local remote-tracking ref of a deleted records branch, or the records fetch prunes.
- `OriginCodeTip` reads no records branch origin no longer holds, so a cancelled slug can run again.
- The refusal's way forward names the real cause, not a code push that will never land.
