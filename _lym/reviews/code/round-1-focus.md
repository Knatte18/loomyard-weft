---
round: 1
focus:
  - "internal/creelengine verbs.go, server.go and the machine-mail files: check that take, report, settle and deliver refuse every sender but `<shortname>:SLACK`, that a settle delivered for a returned message journals once and posts one CREEL notice, and that a second settle writes nothing (D3)."
  - "internal/creelengine scheduler and hour timer: check that a pending machine-outbound message is returned at its hour with the relay down, that no per-hub outbound worker starts for it at the start scan or on a post, and that take never returns a message past its hour (D3)."
  - "internal/creelengine journal fold and retention carry: check that machine-held is written only on a changed reason, and that an older fold reads machine-outbound and machine-inbound events as an id only and folds the held, delivered and returned events to nothing (D3, D8)."
  - "internal/creelengine post refusals: check that D2's edge is exactly orch to orch from a strand with a slack button, that inject, interrupt, hold, release, post --interrupt and forward refuse a `<machine>::` name, and that the own-name check runs before machinename.CheckName (D1, D2)."
  - "internal/slackengine bridgeout.go: check that the bridge-post record keeps an accepted message until creel answers its settle, that refusal counts are separate from orch mail's outbox record and cleared once take stops returning the message, and that D4 holds count toward no cap (D7)."
  - "internal/slackengine bridgein.go: walk the per-message order of D8 and check the operator cursor against the held delivery cursor, the seven-day retention skip, the pin check through bots.info, the hold kept outside retryOrGiveUp, and that an approval's delivery advances the delivery cursor so a revoke holds nothing twice."
  - "internal/slackengine bridge resync, keeper.go and holder.go: check creator and pin tests, that a human-posted envelope never picks the local orch, the once-per-base-interval limit per hub, that a resync rereads no orch channel, and that channel adoption sets the cursors D4 states."
  - "internal/slackengine bridgechannel.go and keeper.go: check name-taken and archived handling (FindChannel at most once per poll ceiling, no unarchive, the way-forward text), the refused-invite retry with `already_in_channel`, operator-change invites, and archiving with the orch's retirement (D4)."
  - "Bridge notice and bridge-post records, the peers record and the bridge map: check they are written through internal/state, tolerate unknown keys, sit in the paths hubgeom builds, and that notice marks are keyed as D4 states and survive a map drop."
  - "approve and revoke in inbound.go and peercommand.go: check case handling, the own-machine refusal in an orch channel, bridge channel routing by peer name and the first-sorted end answering other text, app id kept as written, and the operator-only check (D6)."
  - "The sole-parser and stdlib-only rules: check that no code outside internal/agentname splits `<machine>::`, that no code outside internal/slackengine parses the envelope line, and that internal/agentname imports the standard library alone."
  - "Stamp and authority text (D9, D11): check creelhooks.go, orch-role.md, parent-directive-parent.md and role-slack.md each name `(other machine, trusted orch)` beside the `(other hub, trusted orch)` grant, that docs/slack-app.md gains the pairing text with an unchanged manifest, and that the PATTERN diff lands the declared agent-addressing change."
  - "Test economy: the report lists `internal/hubgeom.TestSlackMachine` as contained, covered by `TestSlackGeometry`; compare their assertions before filing, and judge the new bridge tests for duplication against the existing ones."
---

Seed focus for round 1, drawn from the decision record's D1 to D11 and its acceptance criteria.
The plan format, the answer key's own wording and the redundancy report's unique tests are left out.
