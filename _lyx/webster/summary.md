# Creel stage 1 fixes

All seven batches recorded `status: done` on the first attempt, with no recovery or re-fork.
The plan-level verify gate has not run yet; it runs after this turn ends.

## Batches

- **01-reed-styled-capture** (head `8aaea6fda`): reed's styled pane capture and its `ReedOps` carriage; Claude's styled-to-plain capture with a placeholder-only box blanked.
  Deviations: `claudeengine/session_test.go`, `claudeengine/styledcapture.go`, `shuttleengine/engine.go`, `shuttleengine/fakes_test.go`, `shuttleengine/reed.go`.
- **03-shuttle-box-reads** (head `b9a551990`): shuttle's idle probe, pre-write guard and landing check read the blanked capture, the pane-only probe reports a draft, and Claude declares the restored-prompt clear.
  Deviations: `claudeengine/start_test.go`, `shuttleengine/engine.go`, `shuttleengine/fakes_test.go`.
- **05-shuttle-interrupt-restoring** (head `3ddd99dfc`): shuttle's interrupt for creel clears a prompt its key restored.
  Deviations: `shuttleengine/fakes_test.go`, `shuttleengine/interruptguard.go`, `shuttleengine/run.go`.
- **06-creel-stall-reason** (head `93d40bf45`): the Delivery seam's idle answer carries a stall reason and the engine journals each change.
  Deviations: `creelengine/deliver_test.go`, `creelengine/journal.go`, `creelengine/pool.go`, `creelengine/seam.go`, `testkit/creelfake/creelfake.go`.
  The fork reported its card's `llm`-tier verify line failing in `TestSmokeCreelClaudeStrand`, on Claude Code's folder-trust dialog, and judged it environmental.
  That is the fork's own report, not something I checked.
- **07-creel-stall-wait** (head `45e226fb2`): a waiting inject hears stall changes on stderr and its envelope carries the last reason; creel uses the restoring interrupt, a failed clear counts as sent, and no delivery runs during it.
  Deviations: `creelcli/cli.go`, `creelcli/cli_test.go`, `creelcli/delivery_test.go`, `creelengine/pool.go`, `creelengine/seam.go`, `creelengine/server.go`, `creelengine/server_test.go`.
- **09-inject-ttl-from-posting** (head `ba7f0c1bd`): TTL charged from each inject's own posting; read acknowledges nothing it could not write and shows a returned message's mark; log, help texts, waiting verbs' sender and release match `doc.go`.
  Deviations: `creelcli/cli_test.go`.
- **12-daemon-log-file** (head `67f5426a5`): the daemon's stderr goes to a per-hub log a give-up quotes, the daemon writes its start header and pid-tagged warnings to it, and C1 to C13 join the shuttle sandbox suite.
  Deviations: `creelcli/cli.go`, `creelcli/spawn.go`, `creelengine/daemonlog.go`, `creelengine/server_test.go`.

## Scope guard

Every deviation listed is informational. No batch was failed or held on one.
