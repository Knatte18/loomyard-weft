---
round: 1
focus:
  - "Fidelity, D9: acceptance names `internal/shuttleengine/creelhooks.go` among the passages that grant `(other hub, trusted orch)` its authority; no card file names it. Settle whether some card carries that passage, and whether `orch-role.md`, `parent-directive-parent.md` and `role-slack.md` (card 05) are carried as the Decision states."
  - "Fidelity, D3 and D10: the hour timer in creel's scheduler (armed by a machine-outbound post and the start scan, runs with no relay, returns at the earliest hour) and the take that returns a message past its hour first. Settle which cards carry each, and whether the injected-clock criterion has a planned test that reaches it."
  - "Fidelity, D8: the bridge resync hint, the holder routing for a foreign channel, the per-hub once-per-base-interval limit, and the unpinned-bot notice to the prime orch or each open orch channel. Settle that cards 10 to 12 carry each, and that the hold stays outside `retryOrGiveUp` (card 09 names it)."
  - "Fidelity, D4: the bridge-notice record's keys (send-side by pair, read-side by channel id, kept past a map drop), `FindChannel` at most once per bridge-poll ceiling, and the archived-channel way forward. Settle which card carries each."
  - "Card order: cards 06 to 12 use creel verbs, `PostTimestamped`, `IsDaemonRefusal` and the envelope from cards 04, 05, 07 and 08. Settle that no card depends on a later one."
  - "Attack surface, D2, D3, D6: every new verb, program-only check, `approve` and `revoke` command and routing edge states in its `Intent` what it can let through and what bounds it. Check cards 03, 04, 05, 09 and 11."
  - "Reachability: each refusal a card prescribes (own-name, role, button-off, token-missing, reply-to mismatch, wrong sender program) names an input and a test that reach it, and no upstream check rejects that input first."
  - "Granularity and test economy: cards 01 to 06 and 07 to 12, one card per independently reviewable unit. Check for per-symbol tests where a public-surface test already covers the behavior, and for tests of static prose such as the PATTERN, role or docs passages."
  - "`Edit` cards: member glyphs where `Intent` names symbols, the re-sign arrow where parameters or results change (for example `ParseMachineAddress` taking the machine rule as a function), and `Extends` used on a ref the card rewrites."
  - "`Verify:` lines name no raw test command, and no card leaves a fact to be found out while it runs."
---

Seed focus from reading `00-overview.md`, the decision record and a grep over the card files.
The grep found no card naming `creelhooks`, and the scheduler timer appears only in card 04, so those are the likeliest fidelity gaps.
I did not read the card bodies in full, so the order and attack-surface entries are open questions, not conclusions.
