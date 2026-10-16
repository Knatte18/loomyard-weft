# Lens review

Every lens fork returned a report.

- test-economy, MEDIUM, `--continue` resume asserted in two tables: fixed as proposed — the new top-level test is gone, and its single continue row now lives in `TestStart_AdoptLaunchesResumeSpec` with the no-hint assertion (card 3).
- test-economy, LOW, new top-level start test has the same table shape as `TestStart_AdoptLaunchesResumeSpec`: fixed as proposed — its rows fold into `TestStart_AdoptLaunchesResumeSpec` with want-columns for resume id, prompt source, saved id, hint and `session_id`; the `internal/orchcli` budget raise is dropped (card 3).
- test-economy, LOW, mid-tick rebind row repeats `TestWatcher_StrandReboundMidTickIsNotOverwritten` and needs a new fake hook: fixed as proposed — the row becomes a second case of that test using its `onAlive` hook, and the `onSessionState` fake field is dropped (card 2).
- criteria-coverage, MEDIUM, the adopt prompt is never asserted on a `--continue` launch: fixed as proposed — the adopt and continue rows assert `spec.Prompt` equals `RenderAdoptPrompt` (card 3).
- criteria-coverage, LOW, the adapter's three positional strings could swap the id and the warning with every row green: fixed as proposed — `StartSession` returns a `startedSession` struct with named `GUID`, `SessionID` and `Warning` fields (card 3).
- criteria-coverage, LOW, no row runs `stop` then `--continue` in sequence: fixed as proposed — the row "continue after stop resumes the recorded id" runs `stopCmd` and then `--continue` (card 3); card 4's pointer is updated.
- criteria-coverage, LOW, `internal/orchcli/doc.go` describes neither the record nor the reboot recovery: fixed as proposed — card 3 adds a sentence on the recorded id and `--continue`, pointing at the orchengine `Continuing after a reboot` section.
- forced-edits, LOW, `Session.SessionState` and `logIdleStateDisagreement` comments go stale: fixed as proposed — card 2 lists `internal/orchengine#Session` and `internal/orchengine#Watcher.logIdleStateDisagreement` and rewords both.
- forced-edits, LOW, `wantSessionID` values unstated for most rows and the shared `finished` would change: fixed as proposed — card 1 states each row's value and gives the no-parser row its own `readRun` literal.
- branch-coverage, LOW, the no-recorded-id refusal never sees a state file lacking the key or no state file: fixed as proposed — the row writes raw JSON without `session_id`, and a "continue with no state file" row is added (card 3).
- branch-coverage, LOW, a launch naming no session id is unspecified: fixed as proposed (the "clears on purpose" option) — card 3 states the empty id is recorded and the hint keeps the previous id, with a row for it.
- row-reachability, LOW, the `sinces` re-script in the disagreement test is not forced: fixed as proposed — card 2 leaves that test unchanged, explains why its script still holds, and corrects the ImpactSummary.
- row-reachability, LOW, the new `TestReadSessionStates` rows give no `want` entry: fixed as proposed — the kept new row states its state, cause and since (busy, turn, 07:20) with a turn start after the id-naming turn end (card 1).
- row-reachability, LOW, the fallback-to-record row duplicates "a dead process reads dead": fixed as proposed — that row asserts `recorded` and the new row is dropped (card 1).
- row-reachability, LOW, "before the start lock's work removes any strand" overstates the guards, since `ensureRunsStrand` runs first: fixed as proposed — card 3 says the guards protect the orch strand and the state, and that the live and no-id checks run after `reedUp` and `ensureRunsStrand`, as `--adopt`'s do.
