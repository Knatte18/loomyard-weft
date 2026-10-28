```yaml
lens: bound-breaker
outcome: reported
```

Task: the bound-breaker lens on the creel-agents discussion record, attacking every bound, guarantee and code claim the record makes.

**Claims attacked**
- D1 "the flag adds no privilege; it only picks a receiver the caller could already name": held.
- D1 refusal "when the resolved worktree records no orch strand": held. `OrchStrandOf` answers the fixed name whenever state records a GUID.
- D3 "every wiring points at `orchcli.NotifyParent` or batten's run-notice rule": held. The burler, loom and webster wirings use `NotifyParent`, and batten's `primeShuttle` uses `notifyRunParent`.
- D1/D3 same strand for a held agent: held for pair-run roles. The prime-run tackler and arbiter render no parent and do not ask.
- D2 bound "lym's programs, which only an orch pane or a process outside every pane may claim", with the off-Linux gap stated for orch names only: broken (F1).
- D2/D6 "the same gap already applies to the parent-name trust": held for strand names, broken for program names (F1).
- D8 bound "the operator loses guid addressing" as the only loss, and the rationale that shuttle send "bypasses creel's … pause rules": broken (F2).
- D8 "`creel post --interrupt` covers their uses": held. Pane-mail strands are still reached by `creel interrupt` plus `creel inject`.
- D9 "the expectation follows the holder": held. Forward moves only unread messages, so a read, marked message keeps its holder.
- D10 "the clock counts only while … no other sender's hold stands on it, the rule creel already applies to an inject's TTL": broken (F3).
- D10 "derives the chase state from the journal (read time, `chased` records), so a restart neither repeats nor drops a chase": broken twice (F3, F4).
- D10 "a `post --reply` to a chase … from anyone else is an ordinary reply to the chase": broken (F5).
- D10 bound "a chase carries no sender text, so it cannot pose as an instruction": broken for EXTERNAL senders when the sender is rendered with its envelope (F6).
- D10 bound "a marked message never read is not chased: the reminder backoff and the removed-holder sweep cover it": broken for hook strands (F7).
- D10 "at most two chase posts and one sender notice per marked message": holds only with crash ordering, which the record does not state (F4).
- D10/D12 coverage within the journal's kept window: broken for a frozen clock past retention (F8).
- D13 scan scope (stencils, testdata prompts, Go literals) against the webster allowlist: held.
- D7 "nothing requires peer messaging after D5": held. The only adder is `seatengine/prompt.go`.

**Findings**

F1. MEDIUM · design · D2 Bound (decision-record D2, last paragraph)
D2 widens trust to every lym program's mail but states the identity gap for orch names only.
- Off Linux, `resolveFromEnvironment` takes the program claim whenever the strand name is unset (`internal/creelengine/identity.go:103-110`).
- On Linux, a process detached out of the pane tree is "in no known pane" and its program claim is accepted (`identity.go:77-79`).
- Any lym verb that posts caller-supplied text as a program, such as `lym loom decision add` through `NewProgramClient(target, loomProgram)` (`internal/loomcli/decision.go:909`), then becomes mail every agent is told to act on.

Counterexample: a Codex agent off Linux runs that verb with `LYM_STRAND_NAME` unset. Its text arrives as `&lt;sn&gt;:LOOM` mail, which D2 and D6 say to follow as the operator's instructions.
Fix: the bound names program posing (off Linux, and from a detached process) and what limits it.

F2. MEDIUM · design · D8 Rationale/Bound
`lym shuttle send` does apply the pause rule, and the replacement's other differences go unstated.
- `send.go:45-60` reads the pause record and refuses with the pause's way forward. The rationale says send "bypasses creel's … pause rules", which is false for pause.
- `creel inject` does not refuse under a pause. It queues, freezes the TTL and types the line after the resume (creel `doc.go`, Pause section). An operator's line can therefore land late, after the resume.
- `creel inject` does not append `MessageTail`, which `send.go:56` does through `WithMessageTail`.
- Non-orch strands lose the ability to type into another strand's pane, because inject privilege is limited to EXTERNAL, programs and orch strands.

The bound lists only the loss of guid addressing.
Fix: correct the rationale and state what the replacement now holds rather than refuses, and what it drops.

F3. MEDIUM · design · D10 "the rule creel already applies to an inject's TTL" and the restart sentence
The inject TTL rule does not stop under a hold, and it is not derived from post or read time.
- `ageInjects` adds the elapsed time whenever the worker reads the receiver as running. No hold is checked (`internal/creelengine/deliver.go:555-575`, gated on `w.running`).
- The consumed TTL survives a restart only through `aged` records written every `agedPersistInterval = time.Minute` (`deliver.go:22`, `deliver.go:603-620`).

D10 freezes its clock for pause, dormancy and hold, but derives its state from read time and `chased` records alone.
Counterexample: the holder reads, then a pause covers it for 40 minutes, then the daemon restarts. From the read time the restarted daemon sees more than three intervals elapsed and starts the overdue steps, though the clock should have stood still.
Fix: journal the consumed chase time the way `aged` records do, and correct the claimed parity with the inject TTL.

F4. LOW · design · D10 restart guarantee and the "at most two chase posts" bound
The record names no write order for a chase's `posted` record against its `chased` record.
Counterexample: the daemon journals the chase post and crashes before writing `chased`. The restarted fold sees one chase fewer and posts a third chase, which breaks both "neither repeats" and "at most two".
Fix: one record or a stated order, for example `chased` carried on the chase's own posted line.

F5. MEDIUM · design · D10 "from anyone else it is an ordinary reply to the chase"
`Reply` appends to `original.From` with no receiver check (`internal/creelengine/verbs.go` `Reply`). For a chase, `From` is `&lt;sn&gt;:CREEL`, a program that never reads.
- Program mailboxes are never reminded (creel `doc.go`, Orch strands section).
- Retention carries unread messages forward indefinitely (`journal.go` `retain`).

Counterexample: the holder forwards the unread chase to strand X, and X replies. The reply sits in CREEL's mailbox for good, is re-carried every seven days, and reaches no one. This is a routing edge with no stated bound.
Fix: refuse such a reply with a way forward, or define where it goes.

F6. LOW · design · D10 Bound "a chase carries no sender text"
The chase body "names … its sender". `Sender.String` renders an EXTERNAL sender with its client-claimed envelope: parent process, terminal and working directory (`internal/creelengine/seam.go:159-173`).
Counterexample: an EXTERNAL `post --expect-reply` from a directory whose name holds instruction text. The chase signed `&lt;sn&gt;:CREEL`, which D2 and D6 say to act on, carries that text.
Fix: the chase names the sender by its bare name only.

F7. LOW · design · D10 Bound "a marked message never read is not chased: the reminder backoff and the removed-holder sweep cover it"
The reminder backoff applies only to pane-mail strands. A live, idle `CreelHooks` strand gets one ring and at most one bell per claimable set, never repeated (creel `doc.go`, Idle wake section).
Counterexample: a hook strand that ignores the bell stays live. Its marked message is never read, never chased and never `unanswered`, and a `--wait-reply` sender waits indefinitely.
Fix: state the bound for hook strands, or start the clock at post time when the receiver is a hook strand.

F8. LOW · design · D10 and D12 against journal retention
`retain` carries only unread messages, queued injects and live holds, then prunes everything else (`journal.go` `retain`, `journalRetention = 7 * 24 * time.Hour`).
Counterexample: a read, marked message whose clock stays frozen past seven days (dormant holder or a long pause) is pruned with no `unanswered` event and no answer to the waiting sender.
Fix: carry read-but-owed messages, or settle them at retention.

**Tool calls**
1. grep of creelengine for the receiver rule and identity functions: needed the off-Linux claim path, which the manifest lacked.
2. grep of verbs.go and identity.go functions and of `NewProgramClient` callers: needed the program-claim callers one hop past the record.
3. sed of `resolveSender`, `storePost`/`Reply`, and loomcli's decision notice: needed the reply routing and the program posting path.
4. grep of `ExternalSender` and `Envelope`: needed how an EXTERNAL sender is rendered in a body.
