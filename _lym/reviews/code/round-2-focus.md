---
round: 2
focus:
  - "internal/slackengine/inbound.go `handle` and `forward`: does an orch get marked in `in.awayFrom` only when a Post to it stood, including for an empty body, an oversize message and a give-up after the retry window?"
  - "internal/slackengine/inbound.go `handle` with `awayOnOperatorMessage`: when a first forward fails after turning the presence to slack and the retry succeeds, is the orch woken only once, with no later presence notice?"
  - "internal/slackengine/relay.go presence-notice skip: do the tests cover the notice after an operator message turns the presence and it returns to terminal, a failed forward after the turn, and a set taken by a wake that sends no notice?"
  - "internal/shuttleengine/spec.go `TurnHookCommand` doc against internal/shuttleengine/settings.go merged Stop command: does the doc match the command running the hook at every main-thread turn end?"
  - "internal/slackengine/backoff.go wait reset after a long poll: do the outbound tests pin the reset to the floor and the boundary where elapsed equals the floor or a rate-limit-raised floor?"
---
