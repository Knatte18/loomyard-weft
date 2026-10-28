# Wake the orch after a usage limit resets and hold repeat notices

A provider usage limit stops every session on the account, the orch included.
Until now batten told the orch about each stopped run as an api-error, and each api-error revive typed into the limited session produced a fresh API-error turn end and so a fresh notice.
The orch cannot act on any of them, and after the reset the operator had to tell it to run `lym tender revive`.
This change sends one usage-limit notice per run and limit, holds the orch's mail while its own session is limited, and wakes it once the limit has reset.

Closes #894

## What changes

- `shuttleengine`: `TurnEndActivity` and `AgentActivity` gain a `UsageLimit` value, set when the newest main-chain turn end is a usage-limit API error, with the reset time the limit message states (zero when it states none).
- `claudeengine`: the entry is still recognised only by Claude Code's `rate_limit` classification.
  The reset time is parsed from the text's `resets <time> (<zone>)` clause, with an optional month and day, as the first such instant after the entry's own timestamp in the named IANA zone.
  An unknown zone or a missing clause reads as no reset time.
- `battenshed`: a live run whose newest turn end is a usage limit, idle for the api-error idle time, gets one `usage-limit` notice naming the agent, the reset time and the limit text.
  It goes ahead of the api-error and quiet checks, and no api-error revive is sent for it.
- `orchengine`: when the newest turn end the idle watcher reads for the orch is a usage limit, it records the limit, its reset time and the next wake time in the persisted state, and holds the orch strand's creel deliveries, renewing the hold every tick.
  No cycle starts while the limit stands.
  One minute after the reset time the watcher injects one line rendered from the new orch stencil `orch-template-usage-limit-wake.md`, telling the orch to run `lym tender revive` and check each run.
  It injects again every 30 minutes until the orch answers.
  With no readable reset time the first wake goes 30 minutes after the limit was read.
  A turn end that is not a usage limit clears the record and releases the hold, so the held notices are delivered.

## Design decisions

- The usage-limit notice's episode key is stamped with the reset time and stays sent while any run of the child still reads usage-limited, whatever its activity does.
  It ends when no run reads limited, so a limit that ends and a new one that starts sends a second notice.
  The key holds back only the usage-limit notice: a state change, a dead driver and a plain API error still notice as before.
- One notice per task run stands for one per strand, since a run's agents share the account's limit.
- The orch's own turn end is the reading that counts for its watcher, because the orch shares the limit.
  The hold covers only deliveries to the orch strand and lapses within the cycle hold duration if the watcher dies.
- The wake is a text inject, the watcher's own delivery, which passes its own hold.
  The wake line lives in a stencil read at call time, per the Stencil Ownership Invariant.
  The 1-minute delay and 30-minute retry are constants in `internal/orchengine/usagelimit.go`.
- The new orch state fields are optional, so an old record reads as no limit.
- `lym tender revive` is unchanged: it already skips a strand Claude Code resumed on its own as busy.
- Not covered: a limit read only in a fork's transcript (the main chain's turn end decides), and a hub-wide limit record shared between batten and the orch watcher.
  A limit hit while an orch cycle is in flight is left to the cycle's own timeouts and is read at the next idle turn end.

## To check by hand

- Not run against a live limit.
  If Claude Code words the reset clause differently, the reading falls back to the 30-minute retry; compare the parser in `internal/shuttleengine/claudeengine/usagelimit.go` with a real limit message.
