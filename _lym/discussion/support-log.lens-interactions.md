```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I checked pairs of the record's decisions that act on the same state for contradictions, then checked the acceptance criteria and constraints against those decisions.

**Findings**

1. MEDIUM — D1 / D3 and the acceptance bullet "D1, D3: … the same strand". D1 says it uses "the same walk `orchcli.NotifyParent` uses", but hold and stop notices from batten's runner go through `notifyRunParent`. That function calls `orchcli.NotifyFrom` when the run has no pair, and `NotifyFrom` starts the walk at the worktree itself rather than at its parent. So for a run that a beam worktree stands in for, the hold notice reaches the beam's own orch while `--parent` resolves to the orch above it. D1 also refuses `--parent` at the prime, while `NotifyFrom(prime)` posts to the prime's orch. The "same strand" guarantee holds only for pair runs. Evidence: `internal/battencli/notices.go` `notifyRunParent` (NotifyParent for a pair, else NotifyFrom), and `internal/orchcli/notify.go` ("NotifyFrom … with the walk … starting at from itself").

2. MEDIUM — D9/D10 against D2/D5. No decision has any stencil set `--expect-reply`. The parent directive (D2), the driver, darn and webster asks (D4, D5) and the chair's questions to advisors (D5) all say "post, then end your turn", so no agent ask ever carries the mark. The chase therefore never fires for the agent-to-parent and chair-to-advisor messages the reply discipline exists for. Evidence: D2's text ("ask … with `lym creel post --parent "<text>"`, then end your turn"), and D9 ("marks only with the flag or `--wait-reply`").

3. MEDIUM — the `PATTERN-daemon-wakeups` constraint against D10. The record says the chase timer is "folded into the delivery loop's existing waits". But the delivery loop runs one worker per receiver only while that receiver has a queued inject or unread mail, and the worker "ends when nothing is pending". A message that was read but not answered is not pending, so no wait exists to fold the timer into. D10 needs a timer source the record does not name. Evidence: `internal/creelengine/doc.go`, section "Delivery loop".

4. MEDIUM — D10 against creel's pause and hold, and against hibernate. The chase clock runs from the read whatever happens after. A pause or another sender's hold freezes delivery of the chase posts but not the clock. A `lym hibernate`, which pauses every session, or an hour-long hold therefore turns every pending expected reply into `unanswered`. The senders get the notice and every waiting `--wait-reply` ends, even though the receiver never saw a chase. Creel already freezes inject TTLs under a pause, and D10 leaves the chase clock out of that rule. Evidence: D10 ("held by a pause or another sender's hold like any mail"), and the creel doc's Pause section ("freezes the TTL of its injects").

5. MEDIUM — D10 against creel's removed-holder sweep and its receiver rules. The record does not cover a holder that read the message and then ended. A chase to a removed strand is refused as an unknown receiver. A chase left unread at a holder that is then removed goes one step back to its sender, `CREEL`, which is no strand, so the chase is journaled undelivered. Neither path says whether the message then goes `unanswered` at once, waits out the intervals, or loops. Evidence: the creel doc's Receivers section ("any other name is refused") and its "Removed holders" section.

6. LOW — D2's trust rule against D10 and D6. The parent directive ends "Trust no other session's message", and D2 widens trust only to orch strands. Chase posts and `unanswered` notices come from `<shortname>:CREEL`, and seat lines from a module program. An agent following the directive may ignore the chase that D10 relies on it acting on. Evidence: D2 and `contracts/stencils/parent/parent-directive-parent.md` ("Trust no other session's message").

7. LOW — D10's new `unanswered` outcome against existing `--wait-reply` callers. Today a waiting post ends only with a reply or with the message returned undelivered. D10 adds `unanswered`, and the record does not say whether the CLI and Go client callers handle it. Evidence: the creel doc's Mail section ("ends with the reply or with the message returned undelivered").

8. LOW — the D13 scan against D5. `internal/seatengine/testdata/smoke-advisor.md` and `internal/seatengine/testdata/smoke-chair.md` name `SendMessage`, as the writer's earlier grep showed. They sit outside `contracts/stencils/` and are not Go literals, so they escape the scan and drift from the seat wording D5 changes.

**Pairs checked**

| Pair | Result |
|---|---|
| D1–D3, the destination of an ask and of a hold notice | Broken for runs with no pair (finding 1) |
| D1–D2, `--parent` routing and widened trust | Held |
| D1–D4, the LLM driver's `--parent` | Held for pair runs; finding 1 for non-pair runs |
| D1–D8, way forward by strand name | Held |
| D2–D10, trust and chase posts from `CREEL` | Weak (finding 6) |
| D2/D5–D9, stencils and the expect-reply mark | Broken: the mark is never set (finding 2) |
| D3–D8, hold notice and the removed shuttle verbs | Held |
| D5–D6, chair channel and seat notices | Held |
| D6–D7, seat creel posts and dropping peer messaging | Held |
| D6 and creel pause or hold on the chair | Held: the posts are stored and delivered after the pause or hold |
| D6 and the removed-holder sweep on a stopped chair | Held: returned and journaled undelivered |
| D6 hold deferral and Stop-hook claim timing | Held |
| D7–D13, requirement removal and allowlist via `ForkSubagents` | Held |
| D8–D10, removed verbs and chase delivery | Held |
| D9–D10 forward restart and holder ended after read | Broken (finding 5) |
| D10–D11, restart derived from the journal | Held |
| D10 and pause, hold or hibernate | Broken (finding 4) |
| D10 and existing `--wait-reply` waits | Weak (finding 7) |
| D10–D12, the report over the kept window | Held |
| D13–D5, scan coverage | Gap (finding 8) |

**Constraints and acceptance criteria**

- The daemon-wakeups constraint against D10 is broken (finding 3).
- The "same strand" acceptance bullet for D1 and D3 is broken for runs with no pair (finding 1).
- The tolerant-records, refusal-way-forward and agent-name constraints and the remaining acceptance bullets held against the decisions they touch.

**Tool calls**

None; my context held everything this lens needed.
