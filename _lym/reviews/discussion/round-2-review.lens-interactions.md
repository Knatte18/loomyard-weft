```yaml
lens: interactions
outcome: reported
```

Task: the interactions lens for round 2's review of the discussion pair. I checked pairs of decisions that act on the same state or command.

**Pairs checked**
- D3 (settle and hour) × D7 (bridge-post record, remove an entry once creel answers its settle): held.
- D3 (late settle after return, `CREEL` notice) × D10 (relay down while no button is on): held.
- D3 (take, oldest first) × D7 (holds per message): held.
- D3 (start re-settle before take) × D7 (at-least-once posting): held.
- D7 (`button-off` hold reported) × D10 (relay runs only while some button is on): broken, F4.
- D4 (archived channel dropped, then created or found again) × D4/D7 (`name_taken` adopt, five-refusal cap): broken, F2.
- D4 (adopted channel read from its start) × D8 (dedupe by sender address and id): held for envelopes.
- D4 (adopted channel read from its start) × D6 (operator commands and the "not delivered" answer): broken, F3.
- D8 (held delivery cursor) × D6 (operator messages handled past a hold, answered once): broken, F1.
- D8 (unpinned hold, approval delivers) × D6 (in a bridge channel, approve is handled by the relay whose peer it names): held.
- D6 (approve or revoke of the own machine refused in an orch channel, left to the peer in a bridge channel) × D1 (refusal of own former names): held.
- D8 (skip a `from` naming the own former name, resync accepts the former name) × D1: held.
- D2 (only an orch with a button posts) × D8 (reply to machine-inbound mail journals machine-outbound): held.
- D4 (bridge map holds the peer app) × D6 (approved-peer pin, re-approve by app id): broken, F5.
- D8 (resync-hint rate) × D10 (base interval and ceiling): held.
- D4 (`FindChannel` at most once per ceiling, invite retried each poll) × D10: held.
- D4 (creator check at adopt) × D6 (revoke): held, since D8's pin check gates delivery.
- D9 (stamp by role) × D1/D2 (parser does not judge the role, orch-only enforced at post): held.
- Constraints (`PATTERN-lym-files`, `PATTERN-tolerant-records`, `PATTERN-sole-parsers`) × D4, D6, D7, D5: held.
- Acceptance criteria × Decisions: the D7 `button-off` criterion contradicts D10 (F4); every other criterion I checked matches its Decision.

**Findings**

- **F1. MEDIUM · design · D8 × D6 (`decision-record.md:79`, `:115`, `:169`, `:181`)**
  - The bridge map holds one read cursor per channel, and a held message stops that cursor.
  - D8 also handles operator messages that come after the held message. With only one cursor, every bridge poll reads those operator messages again.
  - So `status`, `revoke` and "not delivered" answers repeat on each poll while the hold lasts. This breaks D6's "answered once". A `button-off` hold can last for hours.
  - The record has no second cursor and no handled-set for operator messages.

- **F2. MEDIUM · design · D4 × D7 (`decision-record.md:84`, `:91`, `:136`; `internal/slackengine/keeper.go:194-197`)**
  - A channel archived by hand keeps its name in Slack, so a create gets `name_taken`.
  - D4's adopt rule checks only the creator. The existing `adopt` refuses an archived channel with a notice to rename it; the record carries neither the refusal nor the notice.
  - Result: the relay adopts the archived channel, the post is refused as archived, D4 drops the channel, and the next send adopts it again. This loops until D7's five refusals return the message, and the operator is never told to rename the channel.

- **F3. LOW · design · D4 × D6 (`decision-record.md:85-86`, `:183`)**
  - An adopted channel, a resync-added one, or one re-found after D4's drop is read from its start.
  - D8's dedupe covers envelope messages only. Operator commands and other operator text already handled in that channel are handled and answered again, which breaks D6's "answered once".

- **F4. LOW · consistency · D7 × D10 (`decision-record.md:133`, `:206`, `:59`, `:260`)**
  - D7 reports `button-off` for pending mail of an orch whose button went off.
  - When that orch was the hub's last button on, D10 stops the relay, so nothing reports the hold. Creel returns the message at its hour as `bridge-unsettled`.
  - The D7 criterion "reports `button-off`" holds only while another orch keeps the relay running, and the record does not say so.

- **F5. LOW · consistency · D4 × D6 (`decision-record.md:79`, `:119`, `:174`)**
  - The bridge map stores each channel's peer app, and the approved-peer record pins the peer's app too.
  - After `approve <machine> <app id>` pins a new app, as for a renamed machine, the map's copy goes stale.
  - The record never says which read uses the map's copy, so the plan writer cannot tell whether it is unused or must be updated on approval.

**Tool calls**
1. I searched `internal/machinename` for `CheckName` and `Names`, to learn the machine-name case rule behind D6's case-insensitive match. The manifest did not include that package.
2. I read `internal/machinename/machinename.go:80-119`, the bodies of `CheckName` and `FromHostname`, for the same reason. Names are lower-case, so I filed no case finding.
