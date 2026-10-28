```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran each decision, D1 to D11, end to end on one concrete case.

**Walkthrough list**
- **D1:** A main-thread `PostToolUse` with one 300-byte post standing; then the same with one 20 KiB agent post. The short post meets the purpose. The 20 KiB post does not (F1).
- **D2:** A Stop with the flag up and mail claimable; then the Stop of a turn woken by D4. Delivery and recording meet the purpose. The count reset does not (F2).
- **D3:** A fork's Bash call while mail stands, then the main thread's Agent tool call returning. Purpose met: the fork claims nothing and the main thread's `PostToolUse` claims.
- **D4:** An idle orch with a live waiter gets a post. The ring wakes it, the model ends the turn, the Stop block claims the mail. Purpose met for the first wakes; repeated idle wakes hit F2. The idle reading D4 names is not what the seam returns (F3).
- **D5:** Interrupted strand, waiter dead, after 5 s. Purpose met for a strand on the new settings. A session started before the deploy gets a bell that cannot deliver (F5).
- **D6:** Agent mail of 20 KiB. Purpose ("in full") missed (F1).
- **D7:** `post --interrupt` to a running shuttle strand: validate, interrupt, store, D5 bell, Stop block. Purpose met. Sent to a hand-added strand, the turn is stopped and nothing is delivered (F4).
- **D8:** A post from `ly:orch` renders `[creel] #1 from "ly:orch": …`; the claim budget measures the quoted line. Purpose met.
- **D9:** A launch line with a deny notice. How the sentence joins the one existing `--append-system-prompt` argument is not stated (F6).
- **D10:** Help on `creel` and `post`. Purpose met.
- **D11:** Docs and stencils. Purpose met.

**Findings**

**F1 — BLOCKING, design.** Location: D1, D6, Constraints ("D1's 8 KiB claim budget keeps each field under the cap").
- A post body may be up to 64 KiB (`internal/creelengine/pool.go:17`, `verbs.go:24`).
- `ClaimAtBoundary` "always hands out the first" message, whatever its size (`internal/creelengine/verbs.go:162` doc, loop at 183–198).
- So a 20 KiB agent post becomes a `reason` and a `systemMessage` of about 20,000 characters each. That is over the 10,000-character cap the record cites, so Claude Code shows a preview in place of each field.
- `hook-read` acknowledges once its write succeeds, so creel journals the message read while the model got a truncated preview.
- Today agent mail is only reminded, and program notices over `maxNoticeBytes` (4096, `reminder.go:23`) are kept out of the line. D6 removes both, so the 4–64 KiB range now goes through the hook unguarded.
- The Constraint's claim is false and D6's "delivered in full" is missed.

**F2 — MEDIUM, design.** Location: D2 count reset, together with D4 rings.
- The record's own fact (Rejected alternatives, "Gating D2 on `stop_hook_active` alone") is that a rewake turn's Stop carries `stop_hook_active` true.
- Walk an idle orch through repeated wakes with no operator turn between them:
  - ring (+1), the woken turn's Stop block (+1), the continuation's real Stop (true, no reset);
  - the count goes 2, 4, 6;
  - no `PostToolUse` claim resets it, because mail never stands at a tool call in those turns.
- After about three idle wakes the cap stops rings, and every later idle delivery goes through D5's typed bell. Under the record's own premises that is the usual path for a mostly idle receiver, not an exception.
- D2 also does not say whether a Stop with `stop_hook_active` false resets the count before or after the cap check.
  - If after: the turn a D5 bell starts reaches its Stop still at 6 and claims nothing.
  - D5 never repeats the bell for an unchanged set, so the mail stays unread until a new post or an operator turn.

**F3 — MEDIUM, design.** Location: D4 and D5, "the strand's idle check through the `Delivery` seam reports no running turn, where a draft in the input box does not count as busy".
- `Delivery.Idle` answers a draft as `Idle:false, Stall:draft` (`internal/creelcli/delivery.go:342–343`). A running turn is `Idle:false` with no stall (`delivery.go:347`), and the probe checks `TooShort` before `Draft`.
- The seam hands the daemon no "running turn" reading. A draft in a too-short pane reads as too-short, and a no-input-box reading may be a running turn or not.
- D4's ring condition therefore cannot be computed from what the seam returns, and the record names no new seam answer.

**F4 — LOW, design.** Location: D7.
- `post --interrupt` to a hand-added strand (`lym reed add`) passes validation, sends the interrupt and stores the post.
- D5 types no bell into a hand-added strand and the strand has no hooks, so its turn is stopped and nothing reaches it.
- D7's Bound ("only where `interrupt` already lets it") does not name this: the stop happens and the delivery the verb promises does not.

**F5 — LOW, design.** Location: D5 bell text, together with the Open risks line on sessions started before the deploy.
- A shuttle-launched session started before the deploy has no `Stop` delivery.
- D5 types it the bell: "End this turn now; creel hands the mail to you as the turn ends." The model ends the turn and nothing is handed over.
- The bell is never repeated for that set, so the mail sits unread with an instruction already spent.
- The Open risk says only that D5 "may bell them", not that the bell delivers nothing.

**F6 — LOW, consistency.** Location: D9, "after the deny notice when there is one".
- The launch and resume lines carry the deny notice as one `--append-system-prompt` argument (`internal/shuttleengine/claudeengine/command.go:254, 357`).
- D9 does not say whether the sentence joins that argument or rides a second flag. A repeated flag may keep only the last value, which would drop the deny notice.

Tool calls:
1. grep of `deliver.go` for the worker's idle gate, and of `settings.go` for the Stop hook's exit setting: the manifest lacked the worker's `Idle` use.
2. `deliver.go:240–275` and the source of `StallDraft`: needed to see how the worker reads idleness.
3. `delivery.go:300–360`, `hubDelivery.Idle`: the manifest lacked how a draft and a running turn map onto `Idleness`.
4. grep of `command.go` and `resume.go` for `--append-system-prompt`: the manifest lacked how the deny notice rides the launch line.
