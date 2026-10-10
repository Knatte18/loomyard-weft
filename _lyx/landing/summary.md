# Bolt and hub stores: forward-only seeding, pull-first writes, quieter gc

Stencil and spec seeding no longer flips Bolt between two binaries' versions, and lyx's other git writes to Bolt and the hub object stores stay correct and quiet.
Bolt took 150 seed commits in three days from two binaries overwriting each other's copies; a body hash says two versions differ, not which is newer.

Closes #477
Closes #478

## What changes

- **Forward-only refresh.**
  The banner stamp of a seeded file now records the writer's revision and commit time beside the body hash.
  `reconcileOne` overwrites an untouched, differing copy only when the running binary is newer than the recorded writer, or when the file records none.
  Newer is decided by `git merge-base --is-ancestor` when the code worktree holds both commits, and by strictly later commit time otherwise.
  A file left alone logs one Info line naming both revisions and `lyx stencil sync` as the override.
  The specs pass follows the same rule.
- **Only a clean production binary refreshes.**
  `buildinfo` gains a production channel, stamped by the deploy tool's production route (the one `update-plugins.sh` runs).
  `stencilstore.Mode` gains an unstamped value, which is the zero value, so an unset mode never refreshes.
  Dev, unstamped and modified-tree builds seed absent files only and warn once per stencil.
  `buildvcs` now reports the commit time.
- **`lyx stencil sync`** ignores the ordering rule and commits as `lyx: sync <subtree>`.
  It records its revision only when the binary could refresh on its own; any other build records none, so the next production binary still refreshes that copy.
- **Seed commits name their binary.**
  The message is `lyx: seed <subtree> (<short revision>[-modified], <channel>, <executable path>)`, built by one label function shared with `hubreconcile`'s hub-wide commit.
- **Pull first, and only when a write is due.**
  The root pre-run classifies first and fetches only if a write is due.
  It then drops seed commits ahead of upstream, fast-forwards Bolt, reclassifies against the pulled copy and commits.
  `hubreconcile`'s hub-wide write and `fabric reconcile`'s hub-config heal run the same pull-first helper.
  A fetch transport failure, or a Bolt still diverged from upstream, skips the writes with one Warn naming the way forward.
  `hubreconcile` leaves its stamp absent when the push fails, so the next start verb reruns the walk.
- **Conflicting seed commits are regenerated.**
  `destroy.go` gains a gated primitive that resets Bolt to its upstream and replays the non-seed commits onto it.
  A seed commit is recognised by its labelled `lyx: seed` subject and by every touched path lying under the stencils or specs subtree.
  When a replayed commit conflicts, Bolt is restored to its starting tip and the push fails as before.
  Bolt's push path runs the primitive after a failed `pull --rebase` and retries once.
- **`lyx fabric reconcile`** exits non-zero, with the envelope intact, when the hub config or warp-binding commit or push fails.
- **Store gc at pair removal.**
  `housekeepStores` skips the gc, keeping the key writes, when reed lists a live session of another pair on the hub's tmux socket.
  The prime's sessions never count, no tmux server counts as none, and any other listing error skips with a Warn.
  The gc passes `gc.pruneExpire=1.day.ago`, so unreachable loose objects older than a day are pruned and the gc stops repacking at every teardown.
  `PATTERN-hub-store-gc` is updated to match; `pattern/PATTERN-leaf-packages.md` and the overview were adjusted for `buildvcs` joining `cliwire`'s allowlist and to bring `PATTERN.md`'s overview back under its size cap.

## Design decisions to know

- Ancestry is exact where the history is present; the commit-time fallback is sound for production builds because each deploy descends from the previous one.
  A machine with a skewed clock can still refresh once in the wrong direction.
- Every stencil Bolt holds today has no recorded revision, so the first production binary after this lands refreshes them once and stamps them.
- A pre-change binary still in use strips the recorded revision when it overwrites a stencil, and the new binary refreshes again; the flip ends when no pre-change production binary runs against the hub.
- Seed or sync commits made by a pre-change binary carry no label and are never dropped, so a push they conflict on fails as today until the operator runs `git pull --rebase` in the board dir.
  The same holds for a hub-wide config commit whose push conflicts: it blocks due pre-run writes until resolved.
- A hub that is never quiet at a removal, or whose probe fails at every removal, is never gc'd by lyx; the manual `git gc` stays the override.
- The one-day expiry newly lets gc delete unreachable loose objects between one day and two weeks old, which git's default kept.
- Every new Warn and refusal names `lyx stencil sync`, `update-plugins.sh`, the board push verb, `git pull --rebase` or a re-run, never a reset or force.

## To check by hand

- Drop not serialised against push rebase (design, MEDIUM, open): in Bolt's push path, check that the seed-commit drop and the retried rebase cannot interleave with another writer's rebase on the same Bolt.
- Forced refresh from a non-production binary freezes the copy (design, MEDIUM, open): `lyx stencil sync` from a dev, unstamped or modified build writes no recorded revision; check the pre-run refreshes that copy under the next production binary.
- The integration suite triage listed `internal/loomcli.TestLoomResumeReconcilesTheHubConfigBeforeArming` as flaky: it failed once and passed on rerun, and was not attributed to this change.
- The deploy tool's production route stamping the channel was not run end to end; run `update-plugins.sh` from a clean pushed `main` and confirm `lyx` reports the production channel.
- The one-day gc prune and the in-flight skip were exercised against the injected probe only, not against a real hub with concurrent runs.
