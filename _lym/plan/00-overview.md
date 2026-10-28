---
format: 5
approved: true
language: go
---

# Plan: creel-agents — every agent messages through creel

Creel stage 3: every agent reaches its parent, its seat peers and the orch through creel, never through Claude Code's `SendMessage`, so a non-Claude engine can take a role that has a parent.
A post can expect a reply; creel chases a holder that read it and did not answer, records the outcome, and `lym report replies` shows which roles need chasing.
`lym shuttle send` and `lym shuttle interrupt` go, and a scan keeps `SendMessage` out of stencils, prompt fixtures and Go message text.

## Card Index

1 — orchroute-walk — move the orch walk, liveness and strand lookups below orchcli into orchroute, with the asker rule
2 — creel-post-parent — add lym creel post --parent, resolved at post time with the asker rule
3 — hold-notice-creel — the hold notice names creel post --reply and creel post instead of SendMessage
4 — notifier-asker-walk — every hold-notice notifier wiring resolves its orch with the asker rule
5 — expect-reply-engine — journal an expected-reply mark on a post, cleared by the holder's reply, shown in the line
6 — expect-reply-cli — post --expect-reply, and the mark and reply-to in read and log output
7 — chase-clock — chase a read, marked, unanswered message twice, then record it unanswered
8 — chase-reply-forward — a reply to a chase answers the chased message; a chase is never forwarded
9 — sweep-holder-gone — a holder gone after reading a marked message settles it unanswered
10 — unanswered-cli-log — post --wait-reply exits 1 on unanswered, and log shows the chase number
11 — report-replies — lym report replies counts expected, replied, chased and unanswered per receiver role
12 — stencils-through-creel — every block 7 stencil site and the seat smoke fixtures message through creel
13 — drop-peer-messaging — remove RequirementPeerMessaging from the closed set and the seat specs
14 — creel-trust-sentence — the appended creel sentence defers other mail to the role's rules
15 — remove-shuttle-send-interrupt — remove the two shuttle verbs and re-point every refusal and doc at creel
16 — agent-messaging-scan — a tier0 scan keeps SendMessage out, and PATTERN gains PATTERN-agent-messaging

## Shared Decisions

### Decision: orchroute-below-orchcli

- **Decision:** the walk, `OrchLive`, `OrchStrandOf` and the orch paths move into a new `internal/orchroute`, which imports only packages already below `internal/creelcli`; `orchcli`'s members become delegations, so no `orchcli` caller changes.
- **Rationale:** `creelcli` cannot import `orchcli`, which imports it (decision record, Constraints); one `orchroute.Asker` serves `post --parent` and every hold-notice notifier, so the two resolve the same orch by construction.
- **Applies to:** cards 1, 2, 4

### Decision: post-options

- **Decision:** `Store.PostWith` with `PostOptions{Wait, Interrupt, ExpectReply}` is the one post entry the daemon calls; `Post`, `PostWait` and `PostInterrupt` stay as wrappers with their signatures.
- **Rationale:** the mark joins the post without re-signing the exported verbs their many test callers use.
- **Applies to:** cards 5, 6

### Decision: chase-clock-cadence

- **Decision:** the chase clock is consumed running time, counted in whole ticks of `agedPersistInterval`: the chaser probes the holder once when it starts and counts that reading as the previous tick's; while the clock runs the chaser ticks once a minute, a tick whose probe and the previous one's both read the holder live, unpaused and free of another sender's hold, one `agedPersistInterval` apart, charges one interval, and a wake between running ticks keeps the tick's deadline; while the clock is stopped each tick doubles the next wait up to `chaseProbeCeiling`, and a running tick or a wake of the holder returns it to one minute; `chase-aged` is journaled at each tick and when the clock stops.
  A step lands at the tick that completes its interval: with the clock running from the chaser's start, chase 1 lands at the 15th tick after the start, and a step after a restart whose time had already passed lands at the 15th tick after the restart, the tick that completes one interval of running time.
- **Rationale:** it mirrors an inject's consumed TTL, so a restart rebuilds the clock from the journal and counts no stopped time (D10); one fixed wait while the clock runs keeps the timer at one wake a minute, and the backoff while it is stopped keeps a probe that runs tmux to `PATTERN-daemon-wakeups`' Backoff rule; both bounds let a test tell the chase wait apart from the probe and sweep waits; a liveness change is seen within a minute while the clock runs, the bound on time charged across it, and a backed-off wait charges no time; a stopped clock's restart is seen within `chaseProbeCeiling`, since a holder coming back live fires no wake, and the running time until then, plus one interval, goes uncharged.
- **Applies to:** cards 7, 8, 9, 10

### Decision: former-holder-reply-guard

- **Decision:** the refusal of a chase reply from a strand that no longer holds the chased message is written as a fail-loud guard with no reaching test.
- **Rationale:** a chase goes only to the holder of a read message, and `Store.Forward` refuses a read message (`internal/creelengine/verbs.go`), so no input moves the chased message after the chase; D10's acceptance row for that refusal has no input that reaches it, and the forward-of-a-chase refusal, which is reachable, carries its own row; the decision record amends D10 and its Constraint so, under "Added after Discussion (parent, 2026-10-28): D10's chase refusals: one way forward, and a fail-loud guard with no reaching test (review F5)".
- **Applies to:** card 8

### Decision: chase-forward-way-forward

- **Decision:** the refusal of a forward of a chase names only the reply route, `lym creel post --reply <chase id>`, which answers the chased message; it leaves out D10's second route, forwarding the chased message itself.
- **Rationale:** the chased message is always read, and `Store.Forward` refuses a read message (`internal/creelengine/verbs.go`), so that route could never succeed, and `PATTERN-refusal-way-forward` asks for a way forward that progresses; the decision record amends D10 and its Constraint so, under the same Added after Discussion entry (review F5).
- **Applies to:** card 8

### Decision: prime-hosted-asker-bound

- **Decision:** `lym creel post --parent` resolves from the caller's worktree, as D1 states, so a strand batten runs in the prime, a tackler or an arbiter, reaches the prime's orch, while batten's own hold-notice wiring keeps the run-notice walk from the run's pair, which D3 already names as the rule and this plan leaves unchanged.
  For a run under a live beam the two can reach different orchs; D1's and D3's acceptance criterion covers agents held in a pair and in a beam, which cards 1, 2 and 4 make agree.
- **Rationale:** routing a prime-hosted strand's question by its run's pair would be a rule the decision record does not state; the divergence is recorded here so a parent can widen D1 if it matters.
- **Applies to:** cards 2, 4, 12

## PATTERN changes

- `PATTERN-agent-messaging` — adds: Messaging another session from an agent: only through creel, never an engine's own tool, bar a fork addressed in-session under ForkSubagents. (test)
