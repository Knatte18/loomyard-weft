---
round: 4
focus:
  - "In `_lym/plan/09-slackengine-relay.md`, the restart cap (step 4 and the `starter.ensure` paragraphs) and its `TestEnsureRelay` and `TestRunRelay` rows: settle whether the spawn count is capped across every way a relay can die, including death after its first accepted read, a healthy relay's re-exec, and a spawn that errors before any relay exists. Trace each path through the writers of `LastSpawnAt`, `NextStartAt`, `FirstReadAt`, `LastSlackOK` and `StartFailures`, and check that each path has a named row that reaches it and that `PATTERN-child-processes` (a retry loop around a spawn caps attempt count) holds."
  - "In `_lym/plan/09-slackengine-relay.md`, any new lifetime rule or constant the cap now uses: settle whether the boundary values (lifetime equal to the constant, `FirstReadAt` equal to `LastSpawnAt`, a zero or earlier `LastSlackOK`) fall in exactly one branch, whether the backoff values the card names are values its rows reach, and whether the outcome of a `whileAway` start before `NextStartAt` is named and handled by card 12."
  - "In `_lym/plan/07-slackengine-inbound.md`, the `away`, `back` and `status` handling and the `@<slug> ` routing: settle whether every error from `Hub.Beams`, `SetPresence` and `ReadPresence` has an assigned cursor outcome and Slack answer, and whether each prescribed guard branch is reached by a named `TestInboundPoll` or `TestSetPresence` row."
  - "In `_lym/plan/03-creel-seams.md` and `_lym/plan/10-slack-cli.md`: settle whether the `ReadWait` contract text agrees with how `Client.CallWatching` in `internal/creelengine/client.go` resumes a lost connection, and whether the `Use` strings card 10 states for `setup` and `relay` pass `useProblems` in the `clitree_test.go` form check."
---

The round 4 plan carries the restart-cap redesign as new text.
The remaining entries recheck the edited sites of the cards named above.
