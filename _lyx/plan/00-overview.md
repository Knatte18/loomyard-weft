---
format: 5
approved: false
language: go
---

# Plan: a pause holds every prompt, verify command and new batch in its scope

A pause holds everything lyx would start in its scope, so the operator never kills processes or re-pauses by hand.
The plan builds the shared pieces first (the pause record's claim helper, the SIGTERM-first verify cancel, `verifytree`'s hold), wires the hold into every verify site, then adds shuttle's typing refusal and makes each of its callers hold, claim or refuse, and ends with creel's interrupt and webster's batch verbs.

## Card Index

1 — pauserecord-claim-session — D6: `pauserecord.ClaimSession` records one session in the live entry covering a slug
2 — verify-cancel-terminates-group — D4: a cancelled `verifyrun.Run` sends SIGTERM to its group, SIGKILL after the delay, and returns once the group is gone
3 — verifytree-pause-hold — D4: `verifytree.Verify` holds before its command, stops it past the grace end and reruns after the resume
4 — paused-marker-readers — D4: a `paused` verify marker reads as no activity and shows as paused in loom status
5 — loomshed-verify-hold — D5: the loomshed verify gate tells the worktree's pause hold
6 — webster-verify-hold — D5: the webster verify gate and `lyx webster verify` tell their pause hold
7 — landing-verify-hold — D5: Merge-In, Finalize and Publish tell a hold a parent lock exempts from slug entries
8 — verified-tree-hold-pattern — PATTERN-verified-tree names the hold, its tripwire checks it, and the overview makes room
9 — shuttle-send-refuses-covered — D1: shuttle's send refuses a covered target with `ErrPaused`, and PATTERN-pause-holds-prompts
10 — shuttle-start-holds-paused-prompt — D2: the run start and the deferred prompt wait out a refusing pause and resend
11 — shuttle-gate-holds-boundary — D3 and D2: the gate loop holds its boundary while covered and holds a paused re-prompt
12 — tender-creel-runner-geometry — D1 and D2: tender's and creel's runners read the hub's pause, and creel holds a paused inject
13 — paused-send-callers — D2: the seat notice sender retries, and `lyx loom resume` and `lyx shuttle send` refuse with the way forward
14 — tender-resume-claims-paused — D2: a resume line refused by another pause claims the session into it
15 — creel-interrupt-claims-covered — D7: `lyx creel interrupt` claims a covered strand, then interrupts it
16 — webster-batch-verbs-refuse-paused — D8: begin-batch and recover-batch refuse under a pause and claim Master

## Shared Decisions

- The PATTERN.md line of `PATTERN-pause-holds-prompts` departs from the declared text, which this plan's `## PATTERN changes` carries verbatim: the declared text is over `pattern.MaxEntryLineChars` (360), which `pattern.Check` and `pattern-entry-line-cap` refuse, and the overview, 15703 bytes at the merge base and past `pattern.HeadroomBytes`, may not grow at all under `patternlint`'s headroom lint.
  Card 9 writes a 262-byte summary line and puts the declared text in full into the entry's background file, and card 8 cuts 313 bytes of trailing clauses from eight entries into their background files, `patternlint.WayForward`'s route, so the overview ends 21 bytes shorter.
  `PATTERN-verified-tree`'s line takes the declared text verbatim.
- `ErrPaused` is a sentinel of its own and `PausedError` carries the judged slug and the covering entry, so a caller waits on the same slug the refusal judged (D2's run start) and an operator verb renders `pauserecord.CoveredError` with its own retry (cards 13).
- `verifytree.NewPauseHold` is the one builder of the hold every site tells, over `pauserecord.ReadCoverage` and `pauserecord.WaitUncovered`, and never returns nil: a geometry with no record reads uncovered, so the drift-guard tests that require every seam filled pin each site's wiring.
- The hub-level tender and creel runners are told the hub's record path with an empty slug, so their sends judge each target by the target's own strand name.
- D7's interrupt is creel's one interrupt step, `Store.interruptTarget`, which `lyx creel inject --interrupt` also runs; card 15 changes that step, so an inject with `--interrupt` under a pause claims and interrupts the strand and queues the inject, which the delivery worker holds until the resume, rather than keeping a refusal D7 retires for the same key.

## PATTERN changes

- `PATTERN-pause-holds-prompts` — adds: Typing a line into a session a pause covers: shuttle's send refuses with `ErrPaused` by the target's own slug, read after the idle wait, and a conditioned send's caller reads coverage itself; of the production runners that type a line, only the orch runner is told no pause geometry and reads uncovered, so `lyx orch start`'s start lines are never held; the caller holds it until the resume or claims the session.
- `PATTERN-verified-tree` — changes: Running a verify command: only via `verifytree.Verify`, told its scope's pause hold. (test)
