# Batten recovers a run whose pair is gone and orders its remote pushes, so no pass fails forever and no push races another

`batten-remote-order` · Task · bug, batten, fabric · running · Code-Burler-Review

Three batten bugs in how a run's pair and its remote branches are ordered and recovered, joined into one task (notes `batten-pass-pair-gone`, `archive-tag-push-race`, `cancel-push-deletes-new-weft`; issue #9).

## 1. A pass fails without end when the task worktree is gone

Seen: a run at Run-Shed whose task worktree no longer exists fails every dobby pass with:

```
battencli: origin reconcile of "<slug>" failed: fabricengine: origin reconcile of run <slug>: no pair worktree <path>
```

The run's status stays `running` with an empty `error` and `hand_back: false`; dobby only reports the repeated failure.
The same came up on 2026-11-03 after `lym fabric remove`, the way forward a stop named, removed a pair but left the failed run's record: the hub's run overview showed two runs of the task, the failed one and the ready entry, and dobby kept failing the old run's pass.

Expected:
- Batten or the origin reconcile recreates or adopts the pair from its branches when they still exist, the way `lym fabric add` adopts an existing weft branch.
- If it cannot, it hands the run back with a stop that names the recovery, instead of staying `running` and retrying.
- A way forward a stop names leaves no run record behind that dobby then drives: a stop that names `lym fabric remove` also clears the run's record, or names `lym batten cancel` instead.

Workaround today: `lym fabric add <slug>` and then `lym batten run <slug>`, or `lym batten cancel <slug>` to clear the record.

## 2. The archive tag is pushed before the weft branch reaches the remote

Seen (2026-11-02, another hub, darn run): Worktree-Teardown failed when the push of the archive tag `archive/<slug>/<sha>` to the weft remote was rejected with "remote rejected ... (missing necessary objects)".
Minutes later `git ls-remote` showed the weft branch at that same commit on the remote, so the commit reached it after the tag push ran.
The tackler then handed the stop back on a false premise, reading the commit as absent from the weft remote.

Expected:
- The archive step waits for, or performs, the weft branch push before it pushes the tag, or retries the tag push on "missing necessary objects" once the branch is on the remote.
- The tackler checks `git ls-remote` before concluding that a commit is missing from a remote.

Workaround today: a hand pass, `lym batten run <slug>`, redoes the teardown.

## 3. The background push of a cancel deletes a new run's weft branch

Seen (2026-11-03): `lym batten cancel <slug> --remove` reported `push_spawned: true`, and the task was released again within a minute.
Dobby started a new run of the same slug, which made a new pair and pushed its weft branch.
At Seed-Child the new run failed at bootstrap because origin no longer had the weft branch: the cancel's background push most likely deleted it after the new run pushed it.
Afterwards origin still held branches of the failed run, which the orch deleted by hand before the task could start cleanly.

Expected: a cancel's background remote work never removes a branch a later run of the same slug owns.
The new run waits until the cancel's push has finished, or the push deletes a remote branch only when it still points at the commit the cancelled run left.

## Common thread

Each case is remote work that runs out of order with the local state or with other remote work.
Discussion decides whether one ordering rule covers them, such as all of a slug's remote work serialised under one lock, or each gets its own fix.
