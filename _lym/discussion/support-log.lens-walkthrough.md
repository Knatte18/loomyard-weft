```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran each decision's mechanism end to end with one concrete case and checked the end state against the decision's purpose. I found four MEDIUM and three LOW findings.

**Findings**

1. MEDIUM — D3 steps 5–6, D2: if the receiver crashes after the confirm, a legitimate message is permanently refused.
   - Case: `ly:orch` posts A#40 to `lyt:orch`. A delivers. B confirms, and A journals `confirmed` and answers `yes`. B dies before it journals its store.
   - A's `deliver` connection fails, so A retries (D2). B finds no duplicate at step 2 and confirms again. A answers `no` because the message is "already confirmed `yes` before".
   - B then answers `refused`, and D2 returns a refused message "undelivered at once". The goal of delivering is missed, and B logs a Warn calling the sender forged.
   - Evidence: D3.5 says yes only once per message. D2 says `refused` returns at once.
   - Possible fix: A may answer `yes` again until it has journaled the message's remote delivery. Or B journals the arrival as pending before it confirms.

2. MEDIUM — D3 step 2 needs a hub path that step 3 has not produced yet.
   - Step 2 matches duplicates on "the same sending hub path and sending id".
   - The `deliver` request carries no hub path. D2 lists its fields, and D3.3 says never to use one the request names.
   - The path is only resolved at step 3, after step 2.
   - Possible fix: resolve first, or key duplicates by sender shortname plus sending id.

3. MEDIUM — D2/D3 set no rule that a daemon must hold no lock while it waits on another daemon.
   - D2 forbids holding the store lock only for the outbound worker's `deliver` call.
   - D3's `deliver` handler makes a `confirm` call to the other daemon while it handles a request. Nothing stops it holding the store lock meanwhile.
   - Case: replies cross. A delivers to B while B delivers to A. Each handler holds its own store lock and calls `confirm` on the other, which needs that lock. Both daemons deadlock.
   - Fix: state the same no-lock rule for the handler's `confirm` call.

4. MEDIUM — D2/D3 give no timeout for the `deliver` and `confirm` calls.
   - The Go client's timeout of zero means none (doc.go, "Call timeout").
   - On a replaced binary, the daemon's `Run` calls `scheduler.Stop()`, which waits for sends in flight, before it closes the listener (`server.go` lines 103–107).
   - Case: a deploy replaces lym for every hub at once. A's in-flight `deliver` waits on B's handler, and B's handler waits on a `confirm` call. If nothing bounds those calls, A's shutdown has no bound.
   - The "retried from one second" in D2 assumes the call returns.

5. LOW — D4: the replier's original is marked replied when the reply is journaled, before the reply is delivered.
   - If D2 later returns the reply undelivered (refused, or after the hour), the original stays marked replied with its chase stopped. The sending side never got an answer.
   - The bound mentions only an unanswered settle and a gone holder, not this case.

6. LOW — D6: "`lym creel read` shows the same stamp after the sender's name" is ambiguous.
   - `read` prints JSON with a `from` field (`internal/creelcli/cli.go` `messageFields`).
   - If the stamp goes into `from`, it breaks consumers that match `from` against names, such as `read --from <sender>` and `reed-slack-relay` D5's stamped-sender filter.
   - Better: state it as its own field.
   - The hook-line claim holds. `MessageLine` renders only `msg.From`, and the EXTERNAL envelope never enters the line, so a claimed envelope cannot imitate the stamp.

7. LOW — D5 acceptance: "`trusted_hubs` is refused in … a machine layer file" may not be testable as written.
   - The config-layers background says the machine layer applies only to a module with a `machine` key.
   - For a Guard-only module, a machine file is not stacked, so it is ignored rather than refused.
   - Fix: word the criterion as "has no effect", or name the check that refuses it.

Outside my lens: `reed-slack-relay` D5 posts mail "whose stamped sender is an orch strand". After this task, `isOrchName` would also match `lyt:orch`, so the relay needs a same-shortname check. That is an interaction for the other lenses.

**Walkthrough list**

- **D1:** The `lyt` daemon starts and records its hub; `ly` looks up `lyt` and finds exactly one hub path. Purpose met.
- **D2:** `ly:orch` posts to `lyt:orch` while B is down. The post is journaled with an id, the worker starts B and delivers, and B answers accepted. Purpose met, apart from findings 1 and 4.
- **D3:** A real delivery succeeds. A hand-written forgery gets `no` and is refused. A crash retry is wrongly refused (finding 1), and step 2 lacks its key (finding 2).
- **D4:** `lyt:orch` replies to B#12, which carries A#40. The reply is delivered to A, confirmed with B, and ends A's wait. Purpose met, apart from finding 5.
- **D5:** `trusted_hubs: [ly]` in `lyt`; a message from `ly:slug:orch` is stored trusted and one from `ly:x:reviewer` untrusted. Purpose met.
- **D6:** The hook-claim line for a trusted message gets its stamp between the quote and the colon, and the EXTERNAL envelope cannot spoof it. Met, apart from finding 6.
- **D7:** A `lyt` reviewer gets trusted mail from `ly:orch`, and the updated parent directive treats it as the delegate. That meets the case seen on 2026-11-05.
- **D8:** An inject from `ly:orch` to `lyt:orch` is refused, naming `post`. Met.
- **D9:** A card files the note through `lym board`. Met.

**Tool calls**

- Grepped `MessageLine` in `reminder.go` and the envelope sites: my context lacked how the line renders the sender, which matters for whether the D6 stamp can be spoofed.
- Read `cli.go` lines 245–314: my context lacked the `read` output shape and how the client fills the EXTERNAL claim.
- Grepped `server.go` for shutdown: my context lacked the drain order on a replaced binary.
- Read `server.go` lines 85–114: to confirm the scheduler stops before the listener closes, for finding 4.
