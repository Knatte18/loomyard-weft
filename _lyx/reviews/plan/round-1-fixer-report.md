# Plan-Review round 1 — fixer report

All nine findings were checked against the code and the decision record, and each premise held.
All nine are fixed.
`lyx loom validate-plan` passes on the fixed plan.

## Fixed

- **F1 (BLOCKING): the socket check ran after the sweep.**
  Confirmed: `sweep` in `internal/testkit/tmuxkit/tmuxkit.go` kills every server and removes each socket file through `removeDeadSocket`.
  Card 7 now creates `afterRun` with a `sweep func()` parameter, and `Main` hands it the real sweep instead of sweeping itself.
  In card 10, `afterRun` runs `checkSockets` first, then `sweep`, then the leftover scan.
  `TestCheckSockets` gains a row that drives `afterRun` over a listening, unregistered socket with a stand-in sweep that removes the file, and asserts the failure, which pins the order.
- **F2 (BLOCKING): reed servers outside reedengine and reedcli were never registered.**
  Confirmed by grep: these packages have no `tmuxkit.Socket` or `KillOnCleanup` call in any tagged file: `burlerengine`, `loomcli`, `orchcli`, `pairteardown`, `seatengine`, `shuttlecli`, `treadleengine` and `webstercli`.
  reedcli's smoke files also register only after `reed status`, which is after the boot.
  Card 6 now states the register-before-boot rule: `reedengine.ServerName(h.Path)` right after the hub is built, or the standalone `lyx-<hash8>` key.
  The registration sits in a `tmux`- or `llm`-only file, never in a helper an `integration` build also compiles.
  Card 6 lists every reed-booting tagged file that lacks a pre-boot registration and adds compile-only `tmux` and `llm` Verify lines.
  Card 10 gains compile lines over every tmux- and llm-tier package, plus a `tmux` run over `pairteardown`, `webstercli`, `standalonegeom` and `tmuxkit`; these catch any file the list missed.
  Card 22 notes that the registration follows a copied hub's path.
- **F3 (BLOCKING): the reed probe was unlicensed scope.**
  Taken to the parent, `ly:orch`, which recorded the amendment "D4: reed's stale-holder probe skips the hermetic test server" in the decision record.
  The overview's `reed-exit-empty-probe` decision and card 5 now cite that amendment and drop the self-licensed "deviation" wording.
- **F5 (MEDIUM): the probe's stated bound was false.**
  The probe now keys on a user option, `@lyx_test_server on`, which only tmuxkit's config sets, instead of `exit-empty off`.
  Card 5's Intent and the overview decision state the true bound: an operator config that sets that option disables the reap for its own servers.
  The kit test asserts the option.
- **F4 (BLOCKING): no card kept PATTERN.md under the cap.**
  Cards 23, 25 and 32 each keep `PATTERN.md` at most 16200 bytes, checked with `wc -c`, and name the clauses they move out of the overview:
  - Card 23 absorbs today's 16380 bytes. It moves four clauses whose background files already state them: from `no-denied-recovery`, `review-round`, `gogit-read-helper` and `hub-store-gc`. It also keeps the leaf entry mergeable with `fswatch`.
  - Card 25 offsets its addition in the same line and drops `comment-line-breaks`'s ", with no column limit", which that line's background file already states.
  - Card 32 creates `pattern/PATTERN-shuttle-stop.md` and moves the per-package reasons there.
- **F6 (MEDIUM): `warplayout_test.go` was classified read-only too early.**
  Confirmed: `lyxcwd.gitWorktreeRoot` spawns `rev-parse --show-toplevel` until card 24.
  The test's move onto `SharedHub` now sits in card 24, after that spawn is gone, with a Verify line.
  Card 19 drops the file and its `SharedHub` use.
  The overview's classification decision, its Applies-to line and the Card Index follow.
- **F7 (MEDIUM): a tagged gate run with no `cmd/lyx` neither set nor stripped the variable.**
  Card 13 now strips the variable on every path that exports no fresh build, and the harness's `prebuildLyx` does the same on an untagged run.
  The Intent states what the variable lets through and its bound.
  `TestGateTest_Scenario` gains a third step: a tagged run in a root with no `cmd/lyx`.
- **F8 (MEDIUM): package comments were left stating false bounds.**
  Card 12 updates `lyxbin`'s package comment and lists `lyxbin.go`.
  Card 7 updates `tmuxkit`'s comment for the private `TMPDIR`, the `/proc` scan and leftover kill, and the exported probes.
  Card 5 updates it for the pre-start, and card 6 for `PackageServer`.
  Card 14 updates `rawGoCommandPattern`'s doc comment.
- **F9 (LOW): the survivors list omitted `MergeHeadPresent`.**
  The overview's `rev-parse-survivors` decision now lists `gitrepo`'s `MergeHeadPresent` as untouched.
  The reason: the `MERGE_HEAD` pseudo-ref is none of the ref reads D12 moves.

## Disputed

None.

## Deferred

None.
