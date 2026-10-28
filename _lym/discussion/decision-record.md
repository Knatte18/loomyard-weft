# Decision record — usage-limit-wake

## Goal

A usage limit that stops the hub's sessions no longer floods the orch with repeat notices it cannot act on, and the orch is woken once the limit resets so it revives what stopped, with no operator heads-up.

## Scope

In:

- The provider-neutral activity reading tells a usage-limit turn end from any other API error and carries the reset time the provider's limit message states.
- The Claude engine reads that from its own classification and parses the reset time from the message text, such as `resets 2pm (Europe/Oslo)`.
- Batten sends one usage-limit notice per task run and limit, naming the reset time, never the api-error notice and never the api-error revive for it.
- The orch watcher reads a usage limit at the orch's own turn end, holds the orch's creel deliveries while the limit stands, and wakes the orch with one stencil-rendered line one minute after the reset, retrying every 30 minutes until the orch answers.
- Docs: the `doc.go` of each touched package.

Out:

- `lym tender revive` is unchanged: it already skips a strand Claude Code resumed on its own as busy.
- Usage limits read in a fork's transcript only; the main chain's turn end decides.
- A hub-wide usage-limit record shared between batten and the orch watcher.

## Decisions

### D1: The activity reading carries a usage-limit reading

`shuttleengine.TurnEndActivity` and `AgentActivity` gain a `UsageLimit` value: whether the newest main-chain turn end is a usage-limit API error, and when it resets, zero when the message states no readable time.
Rationale: batten and the orch both read turn ends through this provider-neutral reading already, and the Claude engine's activity scan already finds the API-error entry the limit marks.

### D2: The Claude engine parses the reset time from the limit message

The usage-limit entry is still recognised only by Claude Code's `rate_limit` classification.
The reset time is parsed from the entry text's `resets <time> (<zone>)` clause, with an optional month and day, against the entry's own timestamp: the first such instant after it in the named IANA zone.
An unknown zone or a text with no such clause reads as no reset time.
Rationale: the operator's idea names the message as the source, and the classification stays the only judge of whether a limit stopped the turn.

### D3: Batten sends one usage-limit notice per run and limit

A live run whose newest turn end is a usage limit, idle for the api-error idle time, gets a `usage-limit` notice naming the agent, the reset time and the limit text, ahead of the api-error and quiet checks.
Its episode key is stamped with the reset time and stays sent while any run of the child still reads usage-limited, whatever its activity does; it ends when no run reads limited.
No api-error revive is sent for it.
Rationale: the repeats came from each revive typing into a limited session, which made a fresh API-error turn end and so a fresh api-error episode.
Bound: the key holds back only the usage-limit notice while the limit stands; a state change, a dead driver and a plain API error still notice as before.

### D4: The orch watcher holds the orch's deliveries while its own session is usage-limited

When the newest turn end the idle watcher reads is a usage limit, it records the limit, its reset time and the next wake time in its persisted state and holds the orch strand's creel deliveries, renewing the hold every tick.
While the limit stands the idle phase starts no cycle.
A turn end that is not a usage limit ends the limit, clears the record and releases the hold, so the held notices are delivered.
Rationale: the orch shares the account's limit, so its own turn end is the reading that counts, and creel's hold already exists for the watcher to keep mail from a session that cannot act on it.
Bound: the hold covers only deliveries to the orch strand, lapses within the cycle hold duration when the watcher dies, and ends at the orch's first turn end that is not a usage limit.

### D5: The wake line is a stencil, injected once per wake time

One minute after the reset time, or 30 minutes after the limit was read when no reset time is readable, the watcher injects one line rendered from a new orch stencil, naming the reset and telling the orch to run `lym tender revive` and check every run.
Each wake moves the next wake 30 minutes on, until the orch answers.
Rationale: the operator's idea; the Stencil Ownership Invariant keeps prompt text out of Go.

## Constraints

- PATTERN-stencil-ownership: the wake line is read at call time through `stencilstore`.
- PATTERN-tolerant-records: the orch state's new fields are optional and an old record reads with no limit.
- PATTERN-shuttle-provider-seam: the message parsing stays in `internal/shuttleengine/claudeengine`.
- PATTERN-daemon-wakeups: the watcher adds no wake of its own; the limit is handled inside its existing tick.
- PATTERN-test-speed: injected clocks; no real waiting.

## Auto-mode assumptions

- One notice per task run stands for "one per strand": a task run's agents share one notice.
- The wake goes as a text inject, the watcher's own delivery, which passes its own hold.
- The retry interval is 30 minutes and the post-reset delay one minute, as constants.

## Open risks

- Claude Code may word the reset clause differently in a later version; the reading then falls back to the 30-minute retry.
- A usage limit hit while a cycle is in flight is left to the cycle's own timeouts and is read at the next idle turn end.

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - batten-usage-limit-precedence-untested (class scope, severity MEDIUM, open in rounds 1)
  - reset-for-fresh-launch-limit-fields-untested (class scope, severity MEDIUM, open in rounds 1)
  - shuttle-activity-usagelimit-copy-untested (class scope, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria

- A usage-limit transcript entry reads as a usage limit with its reset time; a plain API error reads as none; table rows cover a time with and without minutes, with a date, an unknown zone and a missing clause.
- A child whose agent stays usage-limited across several activity changes sends exactly one usage-limit notice, naming the reset time, and no revive.
- A child whose limit ends and a new one starts sends a second notice.
- The orch watcher reading a usage-limited turn end holds the orch strand, starts no cycle, injects the wake line at reset plus one minute, and injects again 30 minutes later while no answer comes.
- With no readable reset time the first wake goes 30 minutes after the limit was read.
- A turn end that is not a usage limit clears the limit and releases the hold.
