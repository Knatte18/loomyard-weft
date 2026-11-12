```yaml
lens: fix-claims
outcome: reported
```

Task: check that the claims the fix added or rewrote hold against the code at HEAD (fix-claims lens).

**MEDIUM: D8 says the removed-holder sweep journals machine-inbound mail as `dropped`, but the code journals swept off-hub mail as `undelivered`.**
- Changed text, D8: "Creel treats machine-inbound mail as off-hub mail for the forward guard, the stamp and the removed-holder sweep. … the sweep returns nothing to another machine and journals such mail dropped."
- Evidence: in `internal/creelengine/sweep.go:239-244`, the sweep's off-hub branch (`len(msg.Path) == 0 && msg.SendingHub != ""`) appends `eventUndelivered` and answers the waiter `OutcomeReturned`.
- `eventDropped` (`internal/creelengine/journal.go:55-56`) is for "mail from another hub that was dropped, never delivered to any agent". Swept mail already reached a holder, so that event does not fit it.
- So treating the mail "as off-hub mail for the sweep" produces `undelivered`, and the same sentence names a different event. The text contradicts itself, and the claim, taken over from the reviewer's F15 fix, does not hold against the off-hub path it models.
- Fix: say the sweep journals such mail `undelivered`, as it does for other-hub mail.

All the other code claims I checked hold:
- D4: `FindChannel` is a paged listing (`api.go:276-308`).
- D4: an `already_in_channel` answer counts as invited, which `API.Invite` already does (`api.go:320`, `ignoreCode(err, "already_in_channel")`).
- D3: off Linux, identity is resolved from the environment and any process can make the program claim (`identity.go:119-129`).
- D8: the existing resync hint rereads every open orch channel with a scan (`inbound.go:358-365`).
- D4 and D8: the adopt-cursor premise holds (`keeper.go:226-233`).
- D7: the outbox record is wiped on a confirmed read (`outbound.go:106-108`).
- D6: command matching lower-cases the whole text (`inbound.go:606`).

Tool calls:
1. Grepped `internal/slackengine` for `already_in_channel` and `Invite`, and read `sweep.go:225-260` and `verbs.go:1025-1045`. My context did not hold the invite code, the sweep branch or the forward guard.
2. Grepped `internal/creelengine/journal.go` for the `eventUndelivered` and `eventDropped` definitions, to see which event the swept off-hub mail gets.
