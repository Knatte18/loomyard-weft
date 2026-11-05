```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: I tried to break each bound, guarantee and claim about existing code in the decision record with a concrete counterexample.

**Claims attacked:**
- D1 "two daemons recording at once both stay": held (rewrite under a lock through `internal/state`).
- D1 "a moved or removed hub drops out": held. The related "addressable after its daemon has started once" is broken (F5).
- D1 refusals for none or several: held.
- D1 / Constraint "no test writes the operator's record": held as far as read. The smoke tests run the daemon in process (`internal/creelcli/smoke_helpers_test.go:113`), so the directory D1 is told applies.
- D2 "`agentname.Parse` reads a foreign name as today": held. `Parse` checks grammar only (`agentname.go:120-138`).
- D2 "an older binary ignores the event whole": held for the fold (`doc.go:64`). Broken for the older binary's retention and its verb handling (F6).
- D2 "exactly once in the receiver's pool": broken (F1).
- D2 the three answers accepted, refused or transient: broken, since a fourth, unclassified refusal exists (F4).
- D2 size check with the widest id: held. The expects-reply form prints the id twice (`reminder.go:38`), and both are covered when each is int64-wide.
- D3 "stops a sender forged with a hand-written `deliver`": held. The confirm checks id, sender, receiver, reply-to and body hash, so a wrong body answers `no`.
- D3 replay bound after the remote delivery is journaled: held.
- D3 "a receiver that dies between confirm and store gets it on the retry": held.
- D3 "between confirm and journal, a replay of the same tuple is stored once by step 3": broken (F1).
- D3 confirm `yes` only for what the sender still vouches for: broken (F2).
- D3 "no store lock across a call, so crossing deliveries both complete": held. The server handles one request per connection, and the confirm takes no lock the other side holds.
- D3 list of what the check does not stop: incomplete (F3).
- D3 / D5 off Linux every message is untrusted: held. Both hubs run on the same machine, so on the same OS.
- D5 "committing the list to its board keeps every change in git": broken (F3).
- D5 exact orch names only: held within the stated reed bound.
- D6 "a stamp after the quote is creel's alone": held. `MessageLine` prints only `msg.From`, never the EXTERNAL envelope (`reminder.go:35-40`).
- D4 "a reply only for an outbound message that went to the replier's hub": held. The receiving side enforces the holder, and the confirm pins the sender.
- D4 "a late reply after retention is refused": held, as stated.
- Constraints facts from `doc.go` (socket name, `SO_PEERCRED`, program claim outside a pane, seven-day retention, ids never reused): held (`doc.go:24,44,49,62,68`; `identity.go:77-78`).

**Findings:**

**F1. MEDIUM, design. D3 steps 3–6 and D2's Bound: the "exactly once" bound breaks under overlapping deliveries.**
- The duplicate check in step 3 and the store in step 6 are not one atomic step, because no store lock is held across the confirm call.
- Counterexample: the sending daemon's 30 s `deliver` times out, or the connection drops, while the receiver's first handler is still in its Strands listing or its 10 s confirm. The sender retries after 1 s. Both handlers pass step 3, both get `yes` (no remote delivery is journaled yet), and both store, so the receiver's pool holds the message twice.
- Fix: step 6 re-checks the (sending hub path, sending id) duplicate under the store lock before it journals, and answers the existing id.
- Evidence: `_lym/discussion/decision-record.md:68-75`, `:60`.

**F2. MEDIUM, design. D3 step 5 against D2's return rules: a confirm still answers `yes` for an outbound message already returned undelivered.**
- The `yes` condition is only "no remote delivery journaled". It ignores the hour expiry and the at-once return after `refused`.
- Counterexample: a deliver that started at T+59:59 sends its confirm at T+60:01, after the sender returned the message. The sender's `--wait-reply` has already ended as returned and the `CREEL` notice has gone out, yet the confirm answers `yes` and the receiver stores the message. The sender may then re-post, and the receiver gets the message twice.
- The same happens when an overlapping retry is still in flight when another attempt is answered `refused`.
- Fix: step 5 answers `no` once the outbound message is returned.
- Evidence: `decision-record.md:70`, `:53-55`.

**F3. MEDIUM, design. D5 and D3's Bound: an uncommitted base-file edit grants trust at once, and neither bound names it.**
- D5 reads `trusted_hubs` with `LoadOrTemplate` at each arrival, from the base file in the board checkout, with no restart. `LoadOrTemplate` reads the layer files on disk (`internal/configengine/config.go:181-184`), not git.
- Counterexample: a process in the sandbox hub `lyt` writes `trusted_hubs: [lyt]` into loomyard's `creel` base file without committing it. The next mail from `lyt:orch` lands in loomyard stamped `(other hub, trusted orch)`. That defeats the one-way trust in the Goal.
- So D5's rationale "keeps every change in git" is false for an uncommitted edit.
- D3's list of same-user routes it does not stop names the journal files, the machine record, the socket, the orch pane and reed, but not the trusting hub's config file.
- Evidence: `decision-record.md:99`, `:103`, `:80`.

**F4. MEDIUM, design. D2 and D3: the receiving daemon's own sender check can refuse a delivery outright, and the record never says how that refusal counts.**
- Every request passes `resolve` before dispatch (`internal/creelengine/server.go:207-210`). A peer in no known pane is refused whenever any live strand's pane process is unknown (`internal/creelengine/identity.go:68-69`). A claim naming a listed strand from outside a pane is refused too (`identity.go:71-75`).
- D2 and D3 do not say whether `deliver` and `confirm` skip this resolution, or what claim a daemon's request carries.
- The refusal is none of D2's accepted, refused or transient answers.
- Counterexample: `lyt` holds a live strand whose pane process is not yet recorded, such as one being added by `lym reed add`. A `deliver` from loomyard's daemon is refused with a generic error. If the sending daemon reads that as `refused`, the message returns undelivered at once.
- Spawned daemons inherit their parent's environment (`internal/creelcli/spawn.go:61-65`, no `cmd.Env`). If a daemon's claim were ever drawn from that environment, a confirm from a daemon that the sending daemon started would hit the second refusal.

**F5. LOW, design. D1: an entry pruned by another hub's daemon is never written back until its own daemon restarts.**
- Any daemon's start drops every entry that fails the shortname check. `fabricengine.ReadShortname` answers not-found on any read error (`internal/fabricengine/shortnamebinding.go:26-29`).
- Counterexample: a live hub whose shortname file is briefly unreadable while another hub's daemon starts is dropped. It stays unaddressable until its own daemon restarts, which can be days.
- D1's Bound says a hub is addressable "after its daemon has started once", which does not hold here.
- Evidence: `decision-record.md:35-36`, `:41`.

**F6. LOW, design. D2: "an older binary ignores the event whole" covers only the fold.**
- An older binary's day-change retention carries only the kinds it knows and then deletes the old files (`doc.go:68`). After a downgrade, pending outbound messages and delivered ones still waiting for a reply are lost silently.
- An older receiving daemon answers the unknown `deliver` verb with a generic refusal. That refusal's class is unstated, the same gap as F4.
- Evidence: `decision-record.md:50`.

**Tool calls:**
1. `internal/creelcli/spawn.go` `spawnDaemon` and the client claim in `goclient.go`: to see what environment a daemon spawned by another daemon inherits, and where a request's claim comes from (F4).
2. `MessageLine` in `internal/creelengine/reminder.go`: to see whether an EXTERNAL envelope reaches the line D6's stamp relies on.
3. Tests that spawn a real creel daemon, and the testkit list: to attack the claim that no test writes the operator's record.
4. Test-environment `XDG` settings and the rest of `goclient.go`: to confirm whether the smoke daemons run in process and how claims are built.
