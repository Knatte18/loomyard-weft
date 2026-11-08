# Round 2 fixer-report — plan

## Changes

- F1, fixed as proposed.
  In card 6, `wireProduction` now fills `newDialer` with a closure that calls `slackengine.NewDialer(baseURL, appToken, &http.Client{}, slackengine.HelloTimeout)`, building the client the way `newSlack` does.
  `plan:internal/slackengine#HelloTimeout` joins card 6's Uses list, since the closure names it.
- F2, fixed as proposed, then narrowed by the self-check.
  - Card 6: a failed `Live` read is wrapped as `slack: read the machine record of hubs <dir>/hubs.json: %w`, which keeps hubregistry's decode refusal and its way forward in the chain.
    A geometry failure names the hub's path and the geometry's own error.
    Both are propagated errors, and the notices of cards 9 and 10 quote them as they are.
    The clause "so no notice sends the operator to move a sound record aside" is gone.
  - Card 9: the `LiveHubs` notice carries the seam's error as it is.
    The test bullet's scripted error carries the hubs record's path, and the notice carries that path.
  - Card 10: the fail-closed notice carries the `LiveHubs` error as it is.
    The `LiveHubs failing` row's scripted error carries the hubs record's path.
- F3, fixed as proposed, then widened by the self-check.
  Card 13's first `TestRunRelay` row gains a relay that has an app-level token and no bot token.
  It mails one notice naming the bot token, `lym slack login` and `lym slack start`, and exits.
  It makes no `chat.postMessage` call in either case: with a map holding the prime's channel, or with no map and a `setup.json` holding `channel_id` `C7`.
  Naming `lym slack start` matches card 13's line 68, which adds that step to every line of the exit.

## Self-check

- fix-consistency: no findings.
- fix-affected-set: no findings.
- fix-claims, MEDIUM, merged with fix-proposal 1, MEDIUM: cards 9 and 10 said the quoted error always names a way forward, but only hubregistry's decode refusal and some geometry errors carry one (`internal/hubregistry/hubregistry.go:146-150`, `internal/hubgeom/creelgeom.go:22`).
  Disposition: fixed with my own change.
  The proposals were to have `liveHubGeometries` add a way forward of its own, or to drop the claim.
  I took the second, because no single next step fits an I/O or permission failure on `hubs.json`.
  Card 6 now says any other failed read carries only the path and the read's own error, and that a geometry error carries a way forward only when that error names one.
  Cards 9 and 10 now say the notice names the way forward when the error carries one.
- fix-proposal 2, LOW: the new card 13 clause seeded a map holding the prime's channel, so the guard on the old `channel_id` post was never reached.
  Disposition: fixed as proposed: the clause now also runs with no map and a `setup.json` holding `channel_id` `C7`.

## Disputed

None.

## Deferred

None.
