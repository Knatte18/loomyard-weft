```yaml
lens: walkthrough
outcome: reported
```

Lens: walkthrough. I ran each decision's mechanism on one concrete case.

**Decisions walked**

- **D1** — The `lyt` daemon starts, records its hub, then looks up `ly`. It finds the loomyard hub path, checks `fabricengine.ReadShortname` on that hub's Bolt directory, and gets one path. **Purpose met.**
- **D2** — `ly:orch` posts to `lyt:orch` while the `lyt` daemon is down. The post is journaled as outbound and the worker builds `hubgeom.CreelGeometry`. `spawnDaemon` starts `lyt` with `cmd.Dir = hubPath` and its log, and the worker sends `deliver`. **Purpose met only in part:** the receiving daemon may answer something other than the three answers D2 names (F1), and retention of outbound messages never ends (F3).
- **D3** — `deliver` from `ly` reaches `lyt`. Steps 1–3 pass, the confirm goes to the `ly` socket and `ly` answers `yes`, so the message is stored and `accepted`. A retry after a lost `accepted` takes the step 3 duplicate path. **Purpose met only in part:** a delivery in flight when the one-hour return fires is confirmed `yes` (F2), and a retry that step 1 refuses reports a stored message as returned (F4).
- **D4** — `lyt:orch` replies with `--reply &lt;lyt-id&gt;`. The stored message holds the `ly` id, so the outbound reply's reply-to is that id. The confirm tuple matches `lyt`'s outbound, and `ly` matches its own outbound to `lyt`, ends the waits and stores the reply. A forward within `lyt` before the reply also works. **Purpose met.**
- **D5** — On Linux, `lyt` with `trusted_hubs: [ly]` receives from `ly:orch` and the exact name passes. A message from `ly:orch-2` is untrusted. **Purpose met.**
- **D6** — Trusted mail to a pane-mail strand: the stamped line fits the sender's size check (the longer "trusted orch" stamp, widest id). The strand gets a reminder with the stamp and no body. **Purpose met.**
- **D7** — The compiled `creelTrustSentence` constant names no shortname, so "own hub" can only mean mail with no stamp. Stamped mail is told apart by the stamp, which D6 puts on the line. **Purpose met.**
- **D8** — `lym creel hubs` from `ly`: the record is filtered by D1, the other socket is probed (the client accepts the answer because the geometry is that hub's), and trust is read from `creel`. **Purpose met.**
- **D9** — A card files the note `creel-cross-hub-trial`. Filing a note is allowed and the task's own entry is untouched. **Purpose met.**

**Findings**

**F1 — MEDIUM, design, D2/D3 (the answer to `deliver` and `confirm`)**
Every request goes through the receiving daemon's `resolve()` before dispatch, and nothing exempts `deliver` or `confirm`. The peer is the other daemon, which sits in no pane. `resolveSender` refuses such a peer with a plain failure whenever a live strand's pane process is unknown. That failure is none of D2's answers (`accepted`, `refused`, transient), and D3 does not say whether it counts as `no` or as a failed confirm. If the worker reads it as `refused`, a message is returned at once for a passing state of the hub.
Evidence: `internal/creelengine/server.go` (`sender, err := d.resolve(conn, req)` before `d.dispatch`, around line 208); `internal/creelengine/identity.go:68-70` and `:129-131`.

**F2 — MEDIUM, design, D2/D3 (the one-hour return against confirm step 5)**
Step 5 answers `yes` when the outbound has "no remote delivery journaled". It does not require the outbound to still be pending.
Case: a `deliver` sent at 59:55 (30 s timeout) is still in flight when the one-hour rule returns the message. The sender gets a `CREEL` notice and its `--wait-reply` ends as returned. The receiver's confirm at 60:05 still gets `yes`, so the message is stored and reaches a trusted orch. The sender, told the message failed, may post again, and the instruction is carried out twice. The same applies to a `refused` that ends the message while a duplicate attempt is still running.
Evidence: decision record D2 (lines 53-55), D3 step 5 (line 70).

**F3 — MEDIUM, design, D2 (retention of an outbound message with no reply)**
Retention carries an outbound message "while it expects a reply and has none". Nothing on the sending side ever settles or ends that state: D2 excludes chase and settle, and D4 says the sending side learns of an unanswered settle or a gone holder only by the missing reply.
Case: the receiver settles the message unanswered after its chases, or its holder is gone. The sending journal then carries the outbound at every seven-day retention, forever. A returned message that expects a reply is carried the same way. The bound D4 states, "a reply arriving after the sending side's retention dropped its message is refused", then never applies to such a message.
Evidence: decision record D2 (line 57), D4 bound (line 93); `internal/creelengine/doc.go:68` (carry only unread or chaseable) and `:284-293` (a chaseable message ends at settle).

**F4 — LOW, design, D3 (step order on a retry)**
Step 1, which applies the receiver rules, runs before step 3, the duplicate check.
Case: a strand receiver stores and reads the message, the `accepted` is lost, and the strand ends. The retry is refused at step 1 because the receiver is no longer listed. The sender's message is returned undelivered with a `CREEL` notice, though it was delivered and read. This breaks "exactly once in the receiver's pool" as the sender sees it.
Evidence: decision record D3 steps 1 and 3 (lines 66-68); `internal/creelengine/doc.go:53` (an unlisted strand is refused as a receiver).

**F5 — LOW, design, D3 (the confirm tuple)**
The confirm tuple has no expects-reply mark, though `deliver` carries one.
Case: a process that can read the sending journal sends the real tuple with the mark flipped before the real delivery arrives. The confirm answers `yes`, and step 3 then answers the real delivery as a duplicate. The receiver ends up holding the message with the wrong mark, so it is chased or not chased against the sender's intent.
Evidence: decision record D2 (line 51, `deliver` fields) and D3 step 4 (line 69, confirm fields).

**Tool calls**
1. Read `internal/creelcli/spawn.go` (lines 1-110) and grepped the claim construction in `goclient.go`, `cli.go` and `client.go`. My context lacked the spawned daemon's environment, working directory and reaping, and how a client's claim is built, which D2's daemon spawn and the `deliver` path depend on.
