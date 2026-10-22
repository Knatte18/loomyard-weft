# Round 1 fixer-report — discussion

Every premise was checked against the code at HEAD and holds.
All changes are in `_lyx/discussion/decision-record.md`, with a round entry and ledger lines Q16 to Q18 in `_lyx/discussion/support-log.md`.

## Changes

- F1, fixed as proposed: D2 splits the owner reading. With the task worktree present, the run-scope reconcile runs. With it absent, the verb does the start check's Bolt pull and owner read and runs no reconcile (`originreconcile.go:574-576`, `owner.go:194-205`). An unreachable pull takes the end-only path, and any other failure refuses. Criterion 1 names the read.
- F2, fixed as proposed (pairteardown option): D5 says `RemovePair` snapshots only locally (`teardown.go:347`) and that a forced `Remove` discards (`remove.go:126-128`). It adds a pairteardown change that snapshots before an origin-mode removal. The Scope line and the Bound are updated.
- F3, fixed as proposed: D5 now has the cancel's removal archive the task branch tip on the warp origin, then delete the branch locally and on origin under its own authority (`remove.go:148`, `:683`, `remoteleftover.go:157-161`). The fabricengine change is added to Scope.
  - Fixed differently in one part: the kept-pair rerun no longer goes through `lyx fabric remove`, because that command keeps the unlanded branch, which the next Add would refuse. It goes through `lyx batten cancel <slug> --remove` instead. The rerun criterion and the notice are updated.
- F4, fixed as proposed (phase-owns option): the cancel phase now owns steps 1 to 7, and batten's lock take runs through an after-end hook. Only the no-pair `End` stays with the verb. The PATTERN text is unchanged and holds.
- F5, fixed as proposed: the reconcile runs with `JudgeOnly` (`doc.go:83-87`, `originreconcile.go:34-37`).
- F6, fixed as proposed: an ended entry that is done refuses, and an ended entry absent from the board goes on with nothing to clear. The self-check reworded both (see below).
- F7, fixed differently: the proposal said an offline `--remove` should refuse before anything is ended. Instead, an unreachable origin makes every cancel end-only. It ends this machine's processes and strands with a Warn. It removes no pair, records or claim and lifts no pause, and it leaves the cancel marker. D2 now states the bound: the stale owner and unread stopped branches are why records and claim wait for origin. Ending the processes is accepted, because an adopted run's driver lives on the owner machine. Refusing outright would leave a live run unstoppable offline, which is the case the operator's revision to Q10 rejected. The criterion is updated.
- F8, fixed as proposed (marker option): D3 adds a cancel marker. It is written before the end and removed last. Batten's pre-run reads it before `armSeed` (`arm.go:377-387`) and refuses. Criterion updated.
- F9, fixed differently: instead of listing pause entries and unpushed commits in D2's "something left" list, or reordering step 9, the F8 marker counts as something left. It outlives every step but the last, so an interrupted cancel's lift, push and notice finish on a re-run. Criterion updated.
- F10, fixed as proposed: on Windows, `runproc.List` runs at D3 step 3 and refuses at once with `taskkill` lines (`end.go:110-112`). Goal and criterion updated.
- F11, fixed with both options: the verb exempts its network gate as `lyx fabric remove` does (`fabriccli/doc.go:56`). Batten's status lock is held only around step 7's records removal, so blocking status readers such as hibernate wait only for that local step. Criterion added.
- F12, fixed as proposed: D6's split moves cleanup's lock-taking out together with its done judgement, and the removal runs under the caller's locks (`cleanup.go:187-206`, `lock.go:27-38`). The plan-writer note matches.
- F13, fixed as proposed: D4 names batten's prime-side driver and run locks (`shedrun.DriverLock`, `shedrun.RunLock`), the pair's own run lock and the pair's driver lock.
- F14, fixed as proposed: D7 says the board saves locally and its detached sync pushes (`board.go:53`). The envelope reports the local write only.
- F15, fixed as proposed: the note names `shedrun.TacklerMarker` under `ScratchDir`, read at D3 step 3.

## Self-check

- fix-consistency, MEDIUM: as written, the marker refusal also covered `lyx batten cancel`, its own way forward. Fixed: the refusal now names `run`, `step`, `goto` and adopt's restart, and the cancel reads the marker itself. Merged with the way-forward and proposal findings below.
- fix-consistency, LOW: the survivor rule named the pair's driver lock, but no phase step probed it. Fixed: phase step 2 now probes it, and the no-pair case skips the probe.
- fix-way-forward, MEDIUM: the marker refusal blocked its own way forward, and the record cited `prepare`, a function that does not exist. Fixed: the record now names `armAt` and exempts the cancel (merged).
- fix-way-forward, LOW: `armAt`'s seed gate would refuse a cancel whose seed is gone. Fixed: the cancel now runs outside that gate.
- fix-claims, MEDIUM: the reconcile reports no reason for an ended entry. Fixed: the verb splits the verdict by reading the entry on the fetched Bolt tip.
- fix-claims, MEDIUM: `refuseStartOwnedElsewhere` writes `claimPrimeOwner` and uses the start wording. Fixed: the cancel reuses only the pull and the owner read, with its own wording. Merged with the proposal finding below.
- fix-claims, LOW: a judge-only reconcile never reports dirty or busy. Fixed: the stopped-branch refusal says so, and the `lyx fabric remove` comparison is dropped.
- fix-proposal, MEDIUM: the landed-ended refusal named `lyx batten run`, but the landing happened on another machine. Fixed: the way forward is now `lyx cleanup --apply`, in a separate bullet.
- fix-proposal, MEDIUM: the start check writes Bolt. Fixed (merged above).
- fix-proposal, MEDIUM: the extra `End` pass ran inside the hook without the write locks. Fixed: the hook reports busy, and the phase reruns step 1 under the write locks.
- fix-proposal, LOW: the marker caught the cancel itself, and two concurrent cancels had no defined outcome. Fixed: a live writer refuses a second cancel, and a dead writer's marker is taken over.

## Disputed

None.

## Deferred

None.
