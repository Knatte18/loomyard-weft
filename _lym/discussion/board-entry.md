# Board entry `teardown-retry-no-pair`: A finished run blocked at Worktree-Teardown with its pair gone can be neither retried nor cancelled

batten run refuses a teardown retry with no pair worktree, though the pair removal already succeeded and only the remote branch delete is left

## Seen (hub orch, 2026-10-31)

Run `pawl-starts-at-preflight` landed on OSL-1046 on 2026-10-30, and its Worktree-Teardown removed the pair but failed to delete the remote weft branch `pawl-starts-at-preflight-weft`: GitHub answered the `git push --delete` with an Internal Server Error.
The run stayed `blocked` at Worktree-Teardown, and its error said "resume this run to retry the deletion".
On Neve a day later, `lym batten run pawl-starts-at-preflight` refused before the retry: `origin reconcile of run pawl-starts-at-preflight: no pair worktree /home/knatte/Code/loomyard-LYMHUB/pawl-starts-at-preflight`.
The pair was already removed, so the way forward the error named can never work, on either machine.

Workaround used: the remote weft branch was deleted by hand (its commit is kept by the tag `archive/pawl-starts-at-preflight/6c0f8a26a141`); ending the run then failed, as below.

## Expected

A Worktree-Teardown retry whose pair is already removed skips the origin reconcile that needs the pair and retries only what is left, here the remote branch deletion, on any machine.
The way forward a teardown error names must be one that works.

## Deadlock (hub orch, 2026-10-31)

After the remote branch was deleted by hand, `lym batten cancel pawl-starts-at-preflight` refused too: "the work of pawl-starts-at-preflight reached its parent; way forward: run lym batten run pawl-starts-at-preflight to finish the landing", which is the verb that refuses for want of the pair.
`lym batten cleanup` removes only done runs' records, so no verb can end the run, and it stays `blocked` at Worktree-Teardown in dobby's overview.
