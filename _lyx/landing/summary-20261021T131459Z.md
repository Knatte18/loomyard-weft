# Origin reconcile, and pushes that never fail silently

A machine no longer acts on a stale view of origin, and a push origin refuses no longer disappears into a trace log.
Two failures drove this: a machine that woke from hibernation drove runs on from stale wefts and Bolt, diverging them from origin, and a read-only account's detached pushes failed with 403 for days with nothing reported outside a WARN in the child's log.

## What changes

- **Origin reconcile** (`internal/fabricengine`).
  Two parallel `git ls-remote origin` calls (warp repo, records repo) compare every branch head with the local tracking refs, then one fetch per repo takes only the moved heads.
  Each branch is judged level, fast-forward, ahead, rewound, rewritten, diverged, gone, dirty, busy or live, and each entry gets one outcome from one classifier: `active`, `ended` or `stopped`.
  An entry that landed elsewhere is torn down locally, keeping any commit only this machine holds under a local `archive/*` tag; a diverged branch is named and stops its entry.
- **Call sites.**
  The reconcile runs at `lyx orch start`, on hibernate wake before the pause is lifted, when batten resumes an existing run, before Merge-In and Finalize, and at the start of `lyx cleanup`.
  `lyx orch start` reports a stop and never refuses.
  An unreachable origin is reported as a warning and every site carries on.
- **Records replay.**
  Local commits on a weft or Bolt are replayed onto origin, dropping those that become empty (by commit on a weft, by board entry on Bolt), and stopping on a real conflict such as a `status.json` both sides changed.
  The reconcile, `Fabric.Pull` and the push retry share it.
- **`lyx fabric pull`** fails with a named way forward when the weft cannot be brought level, instead of returning `ok: true` with `weft_pulled: false`.
  Its warp arm judges unpushed commits and a rewind from the tracking ref's reflog.
- **Rejected pushes.**
  A push is classified: a records-side non-fast-forward fetches, replays and retries once; a bare `(failed)` or transport failure uses the existing backoff; an auth or permission refusal fails at once; a push to a branch gone from origin is never made.
  A push that still fails writes a failed-push record under the hub's board-level `.lyx` directory and sends one creel notice to the orch.
  `lyx cleanup`, `lyx fabric status` and `lyx tender status` print the outstanding records with a way forward per class, and `lyx cleanup --apply` no longer reports a spawned push as landed.
- **Preflight's records push goes detached**, as every other status transition's push is.
- **`internal/pairteardown`** gains a local mode with no origin step, which `run-process-ownership` can reuse.

## Design decisions to check

- The reconcile only moves a branch forward to origin's head or by the records replay; it never resets away a commit origin lacks, never force-pushes and never recreates a remote branch.
- A branch moves only where no live writer can meet it: the prime under a try-acquired prime lock, records branches under their write then push lock (one try-acquire, no wait), a live pair judged but not moved.
- The parent branch is never moved by the run scope; Merge-In's own fast-forward still does that.
- At wake, the daemon's own pause entry is exempt from the network gate for the reconcile's ls-remote and fetch calls only; an entry it does not own is never passed, and the wake reconcile is skipped then.
- Rewound and rewritten code branches are detected from the tracking ref's reflog, so a fetch made outside the reconcile does not erase the evidence.
- `PATTERN-origin-reconcile` and `PATTERN-push-failure-visible` are new PATTERN entries; `PATTERN-fabric-vocabulary` is departed from in one place: the records stop is `recordsStop`, reached through `RecordsStopMessage` and `NewRecordsStopError`, because `battencli` and `battenshed` may not name `fabricengine.WeftStopError`.
- `Landing.Reconcile` is treated as nil-legal in the seam checks.

Closes #737

## To check by hand

Nothing here was run on two real machines against GitHub; the tests use local remotes.
Run `lyx gate test --tags integration ./...` before pushing to `main`.

Findings left open by review, each worth a read:

- `orch-start-error-envelope-bare`: the error envelope of `lyx orch start` carries no reconcile report.
- `publish-not-fast-forward-skips-class-line`: a Publish refused as not fast-forward may skip the class line.
- `rewind-keep-local-merge-noop`: the `ours` merge way forward for a rewound branch with the keep-local choice may be a no-op.
- `tag-created-no-mutation-kind`: the `archive/*` tag creation records no mutation kind.
- `way-forward-line-unasserted-at-print-hops`: the way-forward line is not asserted at every print hop.
- `way-forward-lines-no-directory`: way-forward lines do not name the directory to run them in.
- `way-forward-reentry-verb-shared-string`: the re-entry verb in way-forward lines is one shared string.
- `pattern-entries-rewritten-undeclared`: PATTERN entries other than the two declared were touched; the undeclared ones were restored to the parent's text, so check the diff of `PATTERN.md` and `pattern/`.
- `refused-restore-untagged`: a replay whose restore is refused, or a failed final tip read, must tag the starting tip.
- `watchdog-down-up-idle-window`: the shared reed loop's idle window is derived from the test wait budget; check it on a slow machine.

Residual risks the design accepts until `machine-handover` and `run-process-ownership` land:

- A stale driver on a woken machine that commits only paths the other machine left untouched gets them replayed and pushed; a conflicting `status.json` is what stops it.
- An ended entry's pair is usually busy at wake and stays local until `lyx cleanup --apply` is run twice.
- GitHub's error text is the only signal for the auth class.
