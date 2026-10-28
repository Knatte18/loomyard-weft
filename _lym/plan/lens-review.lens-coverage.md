```yaml
lens: coverage
outcome: reported
```

Lens: coverage. These are defects only; every pair not listed has both a card and a test row.

1. MEDIUM — D2, the Stop exit rule ("exits 0 exactly when it wrote a block, whatever follows the write"). Card 13 states it, but no planned `TestCreelCLI` row writes a block and then has the acknowledgement fail. So the "0 after a failed ack" branch is never reached, and the merged command in card 14 depends on that exit code. Evidence: `_lym/plan/13-creel-hook-verbs.md:33` and `:46-52`.

2. MEDIUM — D4, "a waiter that cannot reconnect exits 0 and never exits 2". Card 13 says an unreachable daemon, a failed reconnect or any refusal exits 0 printing nothing. No row reaches any of these: no unreachable-daemon hook-wait, no refused hook-wait (such as an unknown engine or a non-strand caller), and no hook-wait whose input fails to parse. Evidence: `_lym/plan/13-creel-hook-verbs.md:39`, rows `:50-52`.

3. MEDIUM — D4, "When neither a claim nor a busy reading follows a ring within 30 seconds, the set's next wake is D5's bell". Card 11 counts a ring as answered by a claim or by a probe that reads a turn in progress. Only the claim case has a row. No row has a running turn read after a ring suppress the bell. Evidence: `_lym/plan/11-creel-bell.md:30` and `:44`.

4. MEDIUM — D4 and D7, the criterion "the strand's waiter then rings, and the woken turn's Stop delivers the mail" after the continuation cap. Card 6 pins the cap and card 10 pins the reset on a ring, but no row chains them: Stops up to the cap, a capped Stop that claims nothing, then a ring of the remaining set, then a Stop that hands out mail. Card 18 has no cap scenario either. Evidence: `_lym/plan/06-creel-stop-continuation-count.md:33-36`, `_lym/plan/10-creel-hook-waiter-ring.md:72`, `_lym/plan/18-creel-hooks-llm-smoke.md:20-26`.

5. MEDIUM — the criterion "An idle strand no waiter reaches gets the bell typed once per claimable set and then the message as the Stop block". Lower tiers pin that the bell is typed, but no real-Claude scenario proves a typed bell leads to the Stop-block delivery. Card 18's interrupt scenario does not assert the bell appears once and the message then shows as the Stop block. Evidence: `_lym/plan/18-creel-hooks-llm-smoke.md:25`.

6. MEDIUM — the criterion "A claim at the 8 KiB budget arrives whole in both `reason` and `systemMessage`; what is left goes at the next boundary". Card 12 pins the block JSON only for a short text. Card 13's hook-read rows never claim a budget-sized set and check that both fields hold all the lines and that the rest comes at the next call. Card 18 has no such scenario. Evidence: `_lym/plan/12-shuttle-hook-capability.md:43`, `_lym/plan/13-creel-hook-verbs.md:45-52`.

7. MEDIUM — D4, the no-turn reading when the hook reading decides. Card 7 says a too-short pane or a missing input box keeps `noTurn` true while the reading stays held, which is what keeps them from holding the ring. The only new row is the draft case. No row pins `NoTurn` true for a too-short pane or a missing box under the hook reading, nor `NoTurn` false for a too-short pane in pane-probe mode. Evidence: `_lym/plan/07-shuttle-idle-no-turn.md:23`, `:26` and `:28-29`.

8. LOW — D5, `ErrNotLanded` is never retyped. Card 11 says `ErrNotLanded` records the bell for its set, but its rows cover only nil, a busy send and a draft stall. No row has an unconfirmed send followed by later probes that type nothing more. Evidence: `_lym/plan/11-creel-bell.md:35` and `:40-47`.

9. LOW — D5, the hold rule for the bell. Card 11 says the bell's set under a hold is the owner's mail alone. Only the ring has a hold row (card 10). No row shows a bell staying silent for mail a hold mutes. Evidence: `_lym/plan/11-creel-bell.md:36`, `_lym/plan/10-creel-hook-waiter-ring.md:69`.

10. LOW — D5, the mark "kept through reed's verbatim resume". Card 1 asserts this without a test. `internal/reedengine/strandops_integration_test.go:162` already pins `Provider` surviving a relaunch, and card 1 does not extend that row to `CreelHooks`. Evidence: `_lym/plan/01-reed-creel-hooks-mark.md:17`.

11. LOW — card 5, the split notice's failure branch. When the joined notice is too long and is split, a receiver counts as notified only when every one-entry post succeeds, and the first failure is reported. No row has one of the split posts fail. Evidence: `_lym/plan/05-loom-decision-notice-cap.md:31` and `:36`.

12. LOW — card 4, the boundary of the field cap. The new row uses a 10,001-character body, whose line is already well past 10,000. No row pins that a line of exactly `hookFieldCap` is still claimed, or that the count is in UTF-16 units: an emoji-heavy body under 10,000 bytes but over 10,000 units would be skipped. Evidence: `_lym/plan/04-creel-line-cap.md:40` and `:46`.

13. LOW — card 14, the merged Stop command when the events file cannot be written. The existing `unwritable_events_file` row goes through `recordingHookCommands`, which renders with no flag directory, so the hub's merged command never runs with an unwritable file. Card 14's new test has no such row. Evidence: `_lym/plan/14-claude-creel-hooks-render.md:37` and `:49-53`, `internal/shuttleengine/claudeengine/hookcmd_integration_test.go:33`.

14. LOW — consistency in card 2. Its Intent says "Claude's `Prepare` sets the field in card 13", but card 14 sets `Launch.CreelHooks`. Evidence: `_lym/plan/02-shuttle-launch-creel-hooks.md:18`, `_lym/plan/14-claude-creel-hooks-render.md:44`.

Tool calls:
- Read `internal/fabricengine` `ConfigDirs` and the relaunch rows of reed's `strandops_integration_test.go`. My context did not show whether the config directories card 18 uses set `Board`, which the hooks need (they do), or whether a reed test already pins a field surviving a relaunch.
