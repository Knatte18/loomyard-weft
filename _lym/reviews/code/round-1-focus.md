---
round: 1
focus:
  - "internal/dobbyengine/loop.go, start.go, passes.go: does the worktree-gone check run at the top of every loop iteration, keep State.Tidy only from ticks that end with the git link present, and stop the tidy through the prime-built runner without creating anything at the removed worktree path (Decision 1)?"
  - "internal/dobbyengine start path and internal/beamcli runDobby: is Run's graced lock the only decision, is registerDobby called only after the lock is held, and does the re-exec test (pid plus start ticks, or no ticks) pick the ten-second wait, the full-pass rule in recordStart and the notices as Decisions 2 and 3 say, including the second state read on a refused re-exec?"
  - "Every exit path of Decision 3 (failed exec, refused start, wiring failure, placeDobby refusal, fatal Run error, cancelled context, owned-elsewhere): does each write its dobby.log line, post only where the decision says, and never create a missing log directory?"
  - "dobby.log and dobby.out rotation (Decision 5): only the lock holder rotates, the Windows spawner exception reads Live first, the fd reopen in runDobby happens after placeDobby and before anything else writes, and internal/selfexec keeps its closed import set."
  - "Log policy (Decision 6): no per-pass line, episode-based failure lines, in-memory last-logged run states so a re-exec logs no spurious change, and a heartbeat at start and by the first tick after ten minutes with no new wake."
  - "internal/shedengine, battencli.DobbyLandings, dobbyengine tidy.go: is the machine field tolerant, absent from loom and darn status files, and is batten's wiring refusing the run on a machinename.Name error; is the one-time LandedSinceTidy emptying guarded by the flag and skipped while a tidy is in flight (Decisions 7 and 8)?"
  - "internal/gateslot and hub reconcile: is the adopt seed written only for the hub base gate.yaml, including the absent-file seeding branch, and not by MigrateConfig, the local layer, sparseSeed or run_settings reads (the two Added after Discussion entries)?"
  - "adoptcli check verb, gatecli, beamcli status: lock before any write, suite.log opened by the verb, hubreconcile.Ensure before the gate key loads with a failed reconcile ending the check as failed, per-tier and end notices, tolerant check.json, and the beam status rules for dead pid and differing start ticks (Decisions 10 and 11)?"
  - "Docs and PATTERN: do the doc.go files of every touched package, docs/overview.md and pattern/ changes land in the same diff as the behavior, and does test economy hold (duplicate coverage in loop_test.go, loop_worktreegone_test.go, passes_test.go, tidy_test.go; tier placement of the binary-replacing test)?"
---

Focus set from the decision record's Decisions 1 to 11 and the two Added after Discussion entries, against the diff's package list.
