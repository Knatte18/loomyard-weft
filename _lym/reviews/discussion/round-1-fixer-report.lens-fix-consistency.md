```yaml
lens: fix-consistency
outcome: reported
```

Checked the change set for agreement with sibling decisions, criteria, cross-references and their twins.

**LOW: D2 Bound, the new program-claim sentence**
- **Summary:** The new sentence says a program claim on Linux is accepted "from any process outside every pane", but two lines earlier the same Bound still says lym's programs are ones "which only an orch pane or a process outside every pane may claim".
  - The two lines disagree on the orch pane.
  - The older line also still reads as if it held off Linux, where the new sentence says any process with an unset strand name may claim a program.
- **Evidence:** D2 Bound: "lym's programs, which only an orch pane or a process outside every pane may claim", against the added "A program claim is accepted off Linux from any process whose strand name is unset, and on Linux from any process outside every pane".

**LOW: D10, the new refusal of a reply to a chase by a non-holder**
- **Summary:** D10 adds a refusal with a way forward, but the `PATTERN-refusal-way-forward` constraint still lists only "D1's refusal and every refusal re-pointed by D8".
  - So nothing requires a test to reach the new refusal.
- **Evidence:** Constraints: "`PATTERN-refusal-way-forward`: D1's refusal and every refusal re-pointed by D8 names the way forward, and a test reaches it."

**LOW: D3, re-pointing the burler, loom and webster notifier wirings**
- **Summary:** D3 now changes Go notifier wirings in `burlercli`, `loomcli` and `webstercli`, but the Scope list has no line for this.
  - Scope names only the hold notice's text.
- **Evidence:** D3: "The burler, loom and webster wirings that call `orchcli.NotifyParent` unconditionally change to that rule."
  - Scope: "The parent directive, the hold notice, the LLM driver's notify rules and every other stencil or Go message text that names `SendMessage` move to creel".

**LOW: D11, the reply-to id on log entries (F13)**
- **Summary:** D11 adds a reply-to id on log entries so replies pair with marked messages, but no acceptance row checks it.
  - The D12 row counts "replied" without requiring that the reply be paired with the marked message it answers.
- **Evidence:** D11: "A log entry carries the reply-to id of a reply alongside the mark".
  - Acceptance D11 and D12 rows list only the mark, `chased`, `unanswered` and per-role counts.

**LOW: D10, the widened removed-holder sweep (F1)**
- **Summary:** D10 makes the sweep run "while any such strand exists", which can be for days while a clock is frozen, but the `PATTERN-daemon-wakeups` constraint still bounds only "D10's chase timer".
  - The widened sweep wakeup is not named there.
- **Evidence:** Constraints: "the delivery loop's workers end when nothing is pending, so D10's chase timer is a wait of its own, bounded as D10 states."

**LOW: support-log question ledger, chase clock (F3)**
- **Summary:** The ledger line still says the clock is frozen like an inject TTL, but D10 now says the chase clock also stops under another sender's hold, which an inject TTL does not.
- **Evidence:** `support-log.md:81`: "clock frozen like an inject TTL".
  - D10: "and also under another sender's hold, which an inject's TTL does not."

Tool calls:
- I grepped `support-log.md` and `relevant.md` for mentions of chase, sweep, TTL, `NotifyParent`, kept window and shuttle send. My context did not hold the earlier review-round ledger or the question ledger beyond Rejected alternatives.
