---
round: 2
focus:
  - "internal/creelengine/verbs.go Store.post and Store.Reply: settle whether every local post and reply addressed to the hub's relay program (agentname.FormatProgram(shortname, BridgeProgram)) runs slackRelayCheck, and whether a receiver other than that program skips the bridge entirely."
  - "internal/creelengine/machinemail_test.go TestStore_PostToTheSlackRelay: settle whether a test fails if the receiver guard is dropped, for a post to a local strand and to a beam's program such as ly:zz:SLACK, and whether an EXTERNAL sender row exists."
  - "internal/creelcli/slackbridge.go RelayPostReady: settle whether each read error from geometry, beam listing, forwarding file and relay lock is refused with a way-forward clause, and whether a test reaches it."
  - "internal/creelengine/verbs.go relayDownWarning: settle whether its wording matches the relay's outbound stale-mail age rule in internal/slackengine/outbound.go."
  - "contracts/orchrole/role-slack.md and _lym/landing/summary.md: settle whether their statements about a post to the SLACK program with the button off or the relay down, and about which senders keep their behaviour, match what the code now does."
---

Focus entries restate their own sites; the reviewer still reads the whole range.
