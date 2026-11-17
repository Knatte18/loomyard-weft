```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the fixer's change set agrees with the plan text around it (fix-consistency lens).

Findings:

1. **LOW** — card 6, `**ImpactSummary:**` (`_lym/plan/06-creel-bridge-wiring.md:66`).
   - Summary: the ImpactSummary still says "the new client methods have no caller until card 10", but the fix now says `GoClient.DeliverMachine` has no caller until card 11.
   - Evidence: the new Intent line 44 gives `internal/creelcli.GoClient.DeliverMachine` the reason `no caller until card 11's relay delivers bridge mail`, and card 11 line 70 removes that entry. The ImpactSummary was not updated to match.

2. **LOW** — card 9, the new `TestInboundRead` row "with former name `Old_Box`, `approve old_box` is refused…".
   - Summary: the row needs a hub whose former names include `Old_Box`, but card 9 fixes the test hubs' names and does not say they include it or can be set per row.
   - Evidence: card 9 Intent: "`fakeHub` and `newRelayRun`'s own `Hub` literal with fixed names". No changed text says those fixed names include `Old_Box`.

3. **LOW** — the F6 fix has an unchanged twin in card 11, step 2 of `handle` (`_lym/plan/11-bridge-receiving.md:85`).
   - Summary: in a bridge channel, card 11 still compares the command's machine with this machine's names exactly. Card 9 now compares them case-insensitively.
   - Why it matters: `parsePeerCommand` lower-cases the machine name. So in a bridge channel, `approve old_box` on a machine with former name `Old_Box` is not recognised as naming this machine and is not "left to the peer's relay". It falls through to the other branches.
   - Evidence: card 11 line 85: "an `approve` or `revoke` naming this channel's peer machine runs `runPeerCommand` …; one naming this machine is left to the peer's relay". Card 9 line 40 now says "matches this machine's current or a former name … case-insensitively, as D6 matches a machine name".

Tool calls:
- One grep across `_lym/plan/*.md` for `releaseGoneHolder`, `IsDaemonRefusal`, `no caller`, `case-insensitive`, `runPeerCommand`, `retryOrGiveUp` and `bridgePoll.wake`. My context had only the plan spans I edited, so I needed it to find sibling text such as the overview and the ImpactSummaries.
