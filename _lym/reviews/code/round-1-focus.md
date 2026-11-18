---
round: 1
focus:
  - "D7 in-run relaunch (`internal/shuttleengine`, `internal/reedengine`): does every record write re-read the run record in one lock hold and leave a stop-made terminal record alone, and does a stop at any step leave no strand up?"
  - "Running-run predicate: does every `internal/shuttleengine` site that compared an outcome to `runOutcomeRunning` now go through the one predicate, including attach, carry-over read, orphan sweep, activity and session-state reads, and `StopStepRuns`?"
  - "Relaunch bound: past twice the startup timeout, does an attach or `Wait` end the run as died over a dead strand, and clear the mark and attach over a live one, with `DeliverPendingPrompt` typing an unsent `PromptLine`?"
  - "Relaunch startup wait: does it run with the events proof off and dismiss a trust gate, so a stale session-start line never marks the pane started early?"
  - "Account resolution (D4) and pick (D3): hub pin over beams pin over entry, beams pin yielding only to a future `blocked_until` with an entry, counter reset for added or reweighted accounts, the pick's lock covering the names reader, and no pick for a pinned worktree or the prime."
  - "Config loading (D1): tracked-layer refusal of `accounts` and `account_pins` even when the machine layer holds them, retired or unknown pin refusal, directory comparison after link resolution, and the `CategoryMachine` classification."
  - "Resume transcript carry (D6) and readers (D8): newest copy wins, source stays, live-holder test reads every known directory, the copied flag reaches the new run directory also on a retry after a live-holder refusal, and live readers use the run's launched directory."
  - "Status line verb (D5): window validation, merge order (later reset, higher percentage at one reset), `blocked_until` kept, unlisted name writes nothing, and `OpenLaunch` settings with no creel flag directory."
  - "Orch path (D7): the two `orchcli` capabilities run the relaunch branch body directly without `DecideStart`, spawn no second watcher, and the refused-resume fresh start clears the recorded session id."
  - "Scope against the decision record: the diff's `PATTERN-shuttle-provider-seam` change lands the declared stop-verb relaunch-mode text, the landing notes state the D10 wiring and manual check, and nothing outside the listed scope (other engines, standalone runs) changes beyond the `CLAUDE_CONFIG_DIR` scrub."
  - "Test economy: the redundancy report lists `TestWatcher_RefusedResumeStartsTheOrchFresh`, `TestDispositionCandidate_RelaunchingRecord` and `TestNewForWorktree` as contained; compare each one's assertions with its named covering tests."
---

Focus follows the decision record's riskiest decisions: D7's relaunch state machine and its readers, then D3/D4 resolution, then config, transcript and reader changes.
