---
round: 1
focus:
  - "internal/dobbyengine/runwatch.go runWatch and postWatchLines: trace the state-record read, record, post, record sequence. Settle whether any path repeats, loses or resurrects an owed line or a Watches field, including a failed second record, a restart between post and record, and the whole-map replacement against the writes gateHolds and clearDoneRuns make."
  - "internal/dobbyengine/runwatch.go watchRun and readWatchedRun against D1 to D3: settle the episode stamp, the stops-level comparison and the mtime-gated child read across child absent, failing, vanishing and reappearing, and across a batten state change with unchanged status times. Check each for a missed second stop or a spurious line."
  - "internal/dobbyengine/runwatch.go level and land_notice memory against D2 and D3: when is the level first read relative to the task worktree and its run_settings existing, and what happens to a run first judged past Land. Check the AheadOfLand copy drop for unlisted runs against the landed-line skip."
  - "Which runs reach the watch and the gate check through d.listedRuns (internal/dobbyengine/passes.go, internal/battencli/dobbyreads.go DobbyRuns): settle whether it holds exactly the runs this dobby drives, including runs handed to another machine, held, covered or with an unreadable status, and whether RunView.Run, StatusModified and ChildStatusModified are filled for each."
  - "internal/dobbyengine/donerun.go clearDoneRuns: check the new holds (owed line, unread level, failed record) for a done run that can never be cleared or is cleared before its landed line posts, and check Watches entries that are never dropped."
  - "internal/beamcli/dobbydeps.go RunWatch and GateHolds Post: settle whether the autonomy stamp and the sender are right for a line posted after the task worktree is torn down, and whether the landed line survives teardown as D5 asks."
  - "internal/dobbyengine/gatehold.go gateHolds: settle the once-per-hold key across restart, a failed post, a failed record, a slot set that changes during a multi-slot hold, and a run whose key is pruned or whose entry is dropped and recreated. Check when probeDue is true by the time this step runs in loop.go tick."
  - "internal/dobbyengine/gatehold.go gateHoldBound, gateHoldProcesses and gateHoldLine against D6 and the cap amendment: mixed tiers in one hold, the holder PID absent from the tree sample, marked processes already in the tree, and the cap on each list."
  - "internal/gatecli/run.go: settle whether every acquireSlot caller records a marker that its processes really carry, including non-tier holders and the detached run, and whether gateslot.Holder's doc claim holds."
  - "Tests: check TestRunWatch, TestGateHolds and TestClearDoneRuns against the plan cards' listed rows and the record's acceptance criteria for rows missing or too weak to fail, and judge test economy for the new and changed tests."
  - "Prose the diff changes (contracts/orchrole/orch-role.md Notices, internal/dobbyengine/doc.go and template.yaml, internal/boardcli/cli.go help, docs/overview.md): settle whether each says what the code does and what D7 asks, and whether the boardcli help rewording drops anything."
  - "Commit trailers over the range, in particular the Loosened-Check trailer on the TestTestOnlyCode allowlist entry: settle whether the allowlisting is still present at HEAD and whether any widened assertion in a pre-existing test lacks a trailer."
---

Round 1 seed.
The new engine files `runwatch.go` and `gatehold.go` carry most of the behavior, so most entries point at their state handling.
`_lym/landing/summary.md` was absent at seed time.
