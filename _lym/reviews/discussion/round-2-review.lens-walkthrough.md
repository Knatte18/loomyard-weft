```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran each decision's mechanism end to end, feeding each step the data the step before it produces.

**Decisions walked**

- D1 — `lyt`'s daemon starts, records its hub, and a later lookup of `lyt` from `ly` returns the one path → purpose met.
- D2 — `ly:orch` posts to `lyt:orch` with `--wait-reply`, and the sending daemon queues and delivers → purpose met for delivery. The hour-cap return can cross a stored message (F2).
- D3 — a `deliver` reaches `lyt` from `ly`'s daemon: no pane sender, D1 resolve, duplicate check, confirm, a second duplicate check under the lock, store → purpose met (exactly once in the pool).
- D4 — `lyt:orch` runs `post --reply &lt;lyt id&gt;`. The reply carries `ly`'s id as reply-to, is confirmed by `lyt` and stored in `ly`, and the wait ends → the wait ends, but the stamp is missing from what the waiter sees (F1). The reply's size check is not stated (F3).
- D5 — a message from `ly:orch` arrives at `lyt` with `trusted_hubs: [ly]` on Linux and is stored trusted; `ly:orch-2` is stored untrusted → purpose met.
- D6 — a trusted message gets its hook line, expects-reply form, reminder (`composeReminder`), read header and pane-mail routing; `isNotice` is false for `ly:orch`, so it is reminded → purpose met for the outputs D6 names.
- D7 — the trust sentence and the stamped hook line → purpose met.
- D8 — `lym creel hubs` and the `inject`/`interrupt` refusals → purpose met.
- D9 — a card files the note → purpose met.

**Findings**

F1. MEDIUM · design · decision-record.md D6, with D4's `--wait-reply`
- **Summary:** the cross-hub reply that ends a `post --wait-reply` reaches the waiting strand with no trust stamp.
- **Detail:** in the trial, `ly:orch` reads `lyt:orch`'s answer through `post --wait-reply`. The reply goes back to the waiter through `awaitOutcome` (`internal/creelengine/server.go:420-424`, `resp.Message`). The CLI prints it as `fields["reply"] = messageFields(*resp.Message)` (`internal/creelcli/cli.go:440`). The `returned` and `unanswered` outcomes use the same helper (`internal/creelcli/cli.go:433,437`).
- **The gap:** D6 adds the trust and sending-hub fields only to "the JSON of `lym creel read`", and its criterion checks only `read`. So the record leaves no stamp on the one output the waiting strand reads this mail through. D7's instructions tell the agent to judge mail by a stamp that is absent here.
- **Fix:** D6 states that every rendering of a message from another hub carries the trust and the sending hub, including the outcome of `post --wait-reply` (`reply`, `returned`, `unanswered`). Add a matching criterion.

F2. LOW · design · decision-record.md D2 (hour-cap return) with D3 steps 4–6
- **Summary:** a message can be both stored at the receiver and returned undelivered to its sender.
- **Counterexample:**
  1. The sending daemon's `deliver` call hits its 30-second timeout after the receiver's confirm answered `yes` but before the receiver's store and `accepted`. The receiver spends up to 10 s on the confirm, then takes its store lock and journals.
  2. The call is no longer in flight, so when the hour cap is reached, D2's return goes ahead. Its only guard is "waits for a `deliver` call in flight".
  3. The receiver then stores the message. The sender has already sent a `CREEL` notice saying it was returned undelivered, and no further retry reconciles the two.
- **Evidence:** D2 bullets 4–5 and D3 step 6 in `decision-record.md`; the confirm's `yes` holds until a return is journaled (D3 step 5).
- **Fix:** the record should either bound this, by stating that a return may follow a store when `deliver` times out after a confirm, or close it, for example with a last confirm-free `deliver` probe before the return that a stored message answers by its id.

F3. LOW · consistency · decision-record.md D4 bullet 1, with D2's size check
- **Summary:** for a reply crossing hubs, the record does not say whether the stamped-line size check runs at the reply.
- **Detail:** today `Store.Reply` checks the line with the replier's local name and no stamp: `validateLine(sender.Name, text)` (`internal/creelengine/verbs.go:224-228`). D2 states its stamped check only "at the post". D4 says the reply is "delivered and confirmed by D2 and D3" but does not name the size check.
- **Effect:** a reply near the cap can pass locally, then be refused by the receiver (D3 step 1) and return undelivered, where a local check would have refused it at once.
- **Fix:** D4 states that a reply to another hub is measured as D2's stamped line.

**Tool calls**

1. `sed`/`grep` of `internal/creelcli/cli.go:418-444` and `Store.Reply` in `internal/creelengine/verbs.go`. My context lacked both: the post verb's `--wait-reply` output fields and the reply path's size validation.
