---
round: 3
focus:
  - "decision-record.md D3 step 6 and the D3 flood bound: the operator's 'Added after Discussion (operator, 2026-11-05)' entry requires each dropped delivery to be journaled with sending hub, claimed strand, receiver and reason, and a notice to the receiving hub's orch. Settle whether D3 states the journal entry and the notice, names which orch gets it, bounds the notices under a forged `deliver` flood, and whether the D3 acceptance criterion tests both."
  - "decision-record.md D6 and D7: every place an agent reads a message from another hub (the hook line, the reminder, `lym creel read` text and JSON, and the `reply`, `returned` and `unanswered` fields printed by `lym creel post --wait-reply` through `messageFields` in `internal/creelcli/cli.go`) must carry the trust mark. Settle whether each is covered, and whether the D7 bound is true for sessions launched before the deploy, which keep the trust sentence rendered at launch by `internal/shuttleengine/claudeengine/settings.go`."
  - "decision-record.md D2 return rules and D3 steps 3 and 6: a receiver that stores a message at step 6 while the `accepted` answer is lost (the 30-second timeout, or the sending daemon ending on a binary replace). Settle whether the sender can journal a return for a message the receiver holds, and whether the duplicate key stays correct when a hub's ids restart after its scratch journal is removed or the hub is re-created at the same path."
  - "decision-record.md D3 step 6 and the D2 downgrade bound: how an older binary renders inbound cross-hub mail journaled as an ordinary posted event under the old trust sentence. Settle whether the record states a bound for this exposure."
  - "decision-record.md D4 first bullet: the stamped-line size check applies to a post; settle whether a reply to a message from another hub is measured the same way at `--reply`, since `Store.Reply` measures the local unstamped line."
---

Round 2's fixes are new text; the questions above name where that text must be checked.
