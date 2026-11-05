```yaml
lens: coverage
outcome: reported
```

Task: the coverage lens review of the slack-relay plan (overview and cards 1–14).

Findings:

1. **BLOCKING — `## PATTERN changes` (fidelity to the answer key).** The overview still copies the withdrawn `PATTERN-reap-children` and `PATTERN-slack-boundary` bullets. The parent has since removed both from the decision record's `## PATTERN changes` (creel #766), so `pattern-changes-mismatch` now fails on them. The Shared Decision "PATTERN changes fold into the live clusters" also still describes the old record state. Evidence: `_lym/plan/00-overview.md:56-57` and `:33`.

2. **MEDIUM — D3 criterion "a relay start called from inside a non-orch strand's pane brings up a relay whose first read creel accepts as `<shortname>:SLACK`".** No row drives the double fork.
   - `TestEnsureRelay` only checks the argv of the `--fork` spawn.
   - `SpawnRelayFromIntermediate` has no row.
   - The `relay` verb's `--fork` and `--after` branches have no row in `TestSlackCLI`.
   - So the reparenting that the criterion rests on is never exercised.

   Evidence: `09-slackengine-relay.md:67`, `10-slack-cli.md:62,69`.

3. **MEDIUM — D3 criteria "a prime dobby tick with presence `slack` starts a relay that is down" and "after `lym slack stop` … the next prime dobby tick starts nothing".** Card 12 defers both to `TestEnsureRelay`. The one tick row it keeps, false/nil with no Slack setup, still passes if `hubDobbyDeps` never calls `EnsureRelayWhileAway`. The wiring branch has no failing row. Evidence: `12-dobby-relay-tick.md:17`, `internal/beamcli#TestHubDobbyDeps_PrimeTidyShuttle`.

4. **MEDIUM — D6 "the orch's resume pointer (resume-context and the typed pointer) carries the presence line".** Five sites change: `ResumeContext`, `Watcher.startClearing`, `Watcher.reloadAfterCompaction`, `Watcher.startAutoReload` and `orchCLI.launchOrch`. Rows reach only `ResumeContext` and one watcher reload. The `startClearing` and `reloadAfterCompaction` resume pointers and the `launchOrch` reload pointer have no row, so dropping `WithPresence` at any of them goes unnoticed. Evidence: `11-orch-relay-presence.md:42-44`.

5. **MEDIUM — D3 re-exec branch, "takes its lock again before its next poll".** Step 2's re-exec case has no row. That case is a `RelayState.PID` equal to the relay's own pid, which waits up to 10 s for the lock. The only re-exec row checks that `beforeExec` releases the lock. Evidence: `09-slackengine-relay.md:46,66`.

6. **MEDIUM — D3 `lym slack status` report.** The verb reports presence, relay liveness, pause coverage of the prime orch, channel and operator id, and `LastSlackOK`. The only row checks presence, plus `terminal` with no file. The pause-coverage read and the other fields have no row. Evidence: `10-slack-cli.md:61,69`.

7. **LOW — Card 13 "every return after `EndDobbies`".** It is unclear whether `EndDobbies` returning an error, after it may have ended the relay, counts as "after it". No row covers that `fail` path. Evidence: `13-anchor-relay-restart.md:22,25`, `internal/fabriccli#moveAnchor` (the `ended, err = deps.EndDobbies(...)` error return).

8. **LOW — Card 10 `relay` verb.** `AwaitReparent` errors after its 30 s bound, and no card says what the verb then does: exit, or run anyway. That branch has no row. Evidence: `10-slack-cli.md:62`, `09-slackengine-relay.md:58`.

9. **LOW — D4/D5 Slack answers and posts escape `&amp;`, `&lt;` and `&gt;`.**
   - No inbound row asserts that an answer is escaped.
   - No row covers an answer Slack refuses while the message still counts as handled.
   - No outbound row covers a `5xx` counting toward no cap; only the API test classifies a 500.

   Evidence: `07-slackengine-inbound.md:46,52`, `08-slackengine-outbound.md:32,39`.

10. **LOW — D3 "registered in `internal/runproc` under `runproc.HubOwner` with a role of its own".** No `TestRunRelay` row asserts that the `Register` seam is called. Evidence: `09-slackengine-relay.md:47,66`.

11. **LOW — Card 11 `wireCLI` fills `startRelay` only for the prime.** The planned "beam calls nothing" row uses a nil seam, so the prime-only branch in `wireCLI` goes untested. Evidence: `11-orch-relay-presence.md:34,37`.

12. **LOW — D4 "returns to its floor on … a presence change it reads at each poll".** Card 7 counts only presence changes the poll makes itself, from Slack text, as activity. A change made by `lym slack away` or `back` in a terminal is not stated to reset the wait, and no row covers it. Evidence: `07-slackengine-inbound.md:47`, `09-slackengine-relay.md:49`.

Further tool calls: none.
