# Fixer report — plan, round 1

Every finding's premise held against the code and cards, and all six are fixed.
`lym loom validate-plan` answers `ok: true` after the last edit.

## Changes

- **F1 (BLOCKING), fixed as proposed.**
  Card 9 lists `plan:internal/creelengine_test#TestDaemon_CrossHubDeliver` under `**Edit:**`.
  Its Intent changes the reply-to case of card 7's drop row: `ReplyTo` names a real `outbound` entry of the receiving hub, sent to the sending hub by the delivery's receiver.
  A new `TestDaemon_CrossHubDeliver` line under Rows writes that entry and its `remote-delivered` record as journal lines before `startReceiving`, so no worker starts for it.
- **F2 (MEDIUM), fixed differently in one part.**
  The proposal has `Client.exchange` return a coded refusal as `daemonRefusal`; taken alone, that loses the sentinel the coded refusal carries.
  `codedRefusal` unwraps to `ErrStrandClaimOutsidePane` or `ErrProgramClaimInStrandPane` (`internal/creelengine/identity.go:31`), and `internal/orchcli/notify.go:172`, `internal/loomcli/decision.go:1187`, `internal/slackengine/relay.go:261` and `internal/creelengine/server_test.go:757` match them with `errors.Is`.
  So card 8 has a coded refusal's `daemonRefusal` wrap its `codedRefusal`, and its ImpactSummary says the sentinel holds.
  The rest is as proposed: card 8 gives every other `Client.exchange` failure the code `bad-answer` with no socket text, names such an answer among the transient ones, and gains two `TestScheduler_Outbound` rows.
  On card 7, `listenHubFake` writes the rig's hub path only when the answer leaves `HubPath` empty.
- **F3 (MEDIUM), fixed as proposed, with one addition.**
  Card 9 moves `internal/reportengine/doc.go` from `**Uses:**` to `**Edit:**`, and its Intent states which texts name the `inbound` and `outbound` counts.
  It also lists `internal/reportengine#RoleReplies` under `**Edit:**`, since the field comments change; the review's F6 treats an unlisted changed member as a defect.
- **F4 (LOW), fixed as proposed, with a further bound from the self-check.**
  Card 1's corrupt-record way forward names `lym creel inbox`.
  Card 4's `recordHub` skips the interval after a failed record, its Intent states that a removed file is recorded again past `hubRecordInterval`, and its `TestDaemon_Lifecycle` row asserts both.
  The `d1-way-forward-holds-for-a-live-daemon` decision in `00-overview.md` states the same rule.
- **F5 (LOW), fixed as proposed.**
  Card 5's sentence also names the list of `lym creel` verbs in the **creel** module bullet of `docs/overview.md`.
- **F6 (LOW), fixed as proposed.**
  Card 9 lists `plan:internal/creelengine#Store.storeInboundLocked` under `**Edit:**`.

## Self-check

All four forks returned reports, saved beside this file.

- fix-consistency and fix-claims, LOW: card 4's "the next verb it serves" overstates, since `deliver` and `confirm` skip `recordHub`. Fix as proposed: it reads "the next locally resolved verb".
- fix-claims and fix-proposal, LOW: card 1's way forward and card 4's "holds at once" fail for a hub whose last record succeeded inside the minute. Fix with own change: card 1's way forward says "a minute on", and card 4 gives the interval as the reason, in place of the "holds at once" clause. No change to the interval rule, since making the verb notice a lost entry would read the file on every verb.
- fix-proposal and fix-affected-set, LOW: `ComputeReplies`' doc comment at `internal/reportengine/replies.go:27` goes false too. Fix as proposed: card 9's Intent names it.
- fix-proposal, LOW: the coded row's arbitrary `Code` falls through to the plain refusal at `internal/creelengine/client.go:248`. Fix as proposed: the row names `CodeStrandClaimOutsidePane`.

## Disputed

None.

## Deferred

None.
