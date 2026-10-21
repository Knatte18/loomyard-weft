# Reconcile with origin before acting; never fail a push silently

A machine that was away (hibernation, another machine's work) no longer acts on a stale view of origin, and a push origin refuses no longer disappears into a trace log.
On 2026-10-19 a woken machine drove runs on while its weft and Bolt had diverged from origin, and its rejected pushes went unreported.
Separately, a read-only account made every detached weft and Bolt push fail with a 403 for three days without a visible sign.

Closes #737

## What changes

**Origin reconcile.**
`internal/fabricengine` gains one reconcile that makes two parallel `git ls-remote origin` calls (warp repo, records repo) and fetches only the heads that moved.
It judges each branch as level, fast-forward, ahead, rewound, rewritten, diverged or gone, and each entry as `active`, `ended` or `stopped`.
It runs before the steps where a stale view does harm:

- `lyx orch start` prints the report and the outstanding failed pushes and passes both into the orch's start note; it never refuses.
- The hibernate daemon's wake runs it before lifting its pause entry.
- Batten resume, Merge-In and Finalize run the run-scope form; `ended` and `stopped` are refusals or a Stuck naming the way forward.
- `lyx cleanup` runs it first; without `--apply` it is judge-only and moves no ref, with `--apply` a Bolt or prime-weft stop refuses the verb.

An entry that landed or was removed on origin is ended locally: its pair is removed through a new local mode of `internal/pairteardown` that runs no origin step and no board write, and unpushed work is kept under a local `set-aside/<slug>/<sha>` tag.
Origin unreachable is the outcome `unreachable`, warned, and the caller carries on; wake retries within a bound first.

**Records replay.**
A replay of local records commits onto origin drops commits that become empty, and sets aside commits of ended entries.
On a weft it cherry-picks and stops only on a real conflict, such as a `status.json` both sides changed.
On Bolt it replays by entry through `internal/boardengine`, so two machines changing different entries never conflict; where both changed the same field, origin's value wins and the lost value is kept under a `set-aside/` tag and reported.
`Fabric.Pull`, the push retry and Bolt's pull-first share it.
`lyx fabric pull` now fails with a way forward when the weft cannot be brought level, instead of `ok: true` with `weft_pulled: false`; the manual `reset --hard` recovery is gone from its doc.

**Visible push failures.**
A rejected push is classified.
A records-side not-fast-forward fetches, replays and retries once; a held write lock defers (`deferred` record, no mail).
Auth refusals fail at once, and a branch gone on origin is never recreated.
A push that still fails writes a failed-push record under the hub's `.lyx` directory and sends one creel mail to the orch, signed `FABRIC`.
`lyx fabric status`, `lyx tender status` and `lyx cleanup` print outstanding records, and `lyx cleanup --apply` no longer reports a spawned push as landed.

The Preflight row's records push is now detached, as every other status transition's push.

New invariants `PATTERN-origin-reconcile` and `PATTERN-push-failure-visible` are in `PATTERN.md`.
Module docs are updated in `internal/fabricengine/doc.go` and the other touched packages.

## Design decisions to know

- A branch moves only where no live writer can meet it: busy prime, locked records branches and live pairs are judged and reported, never moved; locks are try-acquired with no wait.
- Code branches are never rebased or reset away; a branch rewound on origin by another machine stops its entry and is not restored by the next push.
  The keep-local way forward is `git push origin <branch>` in the task worktree, because origin's head is already an ancestor of the local tip and an `ours` merge would change nothing.
- The reconcile reads rewind evidence from the tracking ref's reflog, so it relies on `core.logAllRefUpdates` and on entries outliving the reflog expiry.
- Bolt conflicts keep origin's value rather than stopping, since every Bolt push would meet the same replay and a stop would have no way forward.
- Seams (live-pair probe, teardown, notify, Bolt entry merge) are injected, since `pairteardown`, `orchcli` and `boardengine` import `fabricengine`.
- `Landing.Reconcile` is nil-legal and loom smoke runs keep the detached board sync from pushing.
- `gitrepo` returns a push auth refusal as its own class, and forces the pack index rebuild under the exclusive lock so concurrent readers never see a half-built index.
- Out of scope: owners and takeover (`machine-handover`), ending orphaned processes (`run-process-ownership`), async remote operations (`fabric-remote-async`), dobby's calls (`umbrella-daemon`).

## Check by hand

Nothing here was run on two real machines against a real origin.

- Wake on a machine with no network, and wake under an operator's hub pause (the reconcile is skipped, the notice names the entry).
- A 403 on a detached weft push: one record, one mail, and the record outstanding in `lyx fabric status`.
- `lyx cleanup` without `--apply` against a diverged Bolt, then with `--apply`.
- `PATTERN.md` rewrites entries beyond the two new ones (`PATTERN-durable-vs-ephemeral-state`, `PATTERN-hub-suffix`, `PATTERN-push-both-sides`, `PATTERN-recipe-setting-lookup`, `PATTERN-edit-directive`) and adds `pattern/PATTERN-recipe-setting-lookup.md`; these were not declared in the design and need a decision.
- `internal/gitrepo/gogit.go`: the pack index rebuild is forced under the exclusive lock; check that concurrent readers cannot still see a half-built index.
- `orch start` error envelope carries no report when the reconcile itself errors.
- A Publish not-fast-forward does not print the class line the other push paths print.
- The keep-local rewind way forward (a plain push) is worded the same in every refusal that names it.
- The set-aside tag creation has no mutation kind in the `*Mutations` record.
- Way-forward lines are not asserted at the print hops of status, tender and cleanup, and they share one re-entry verb string.
