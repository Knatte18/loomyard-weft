```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I checked every pair of decisions in the record that share state, then each acceptance criterion and constraint against the decisions it touches.

**Pairs checked**
- D1 × D2 (receiver lookup at post): held.
- D1 × D3 step 2 (sender lookup; none or several refused, and a refusal returns the message at once): held, matching D1's bound.
- D1 × D5 (trust by shortname, which must be unique by D1): held.
- D2 × D3, the confirm answering `yes` only while the message is pending, against D2's return rules: held.
- D2's hour return × D3 step 6 store: **broken**. A store and a return can both happen (F2).
- D2's lock-free calls × D3 step 5 confirm served while a `deliver` is in flight: held.
- D2's downgrade rule × D3 step 6 (inbound mail journaled as a `posted` event) × D7: **broken** (F3).
- D2 size check × D6 stamp × D3 step 1 receiver cap: held. The sender measures the widest stamp and id, so a line it passes always passes the receiver.
- D2 retention × D4 late reply: held, and stated as a D4 bound.
- D2 × D8 (`--interrupt` refused, `--wait-reply` and `--expect-reply` kept): held.
- D3 step 3 duplicate check × D4 reply rule as a receiver rule applied after it: held.
- D3 step 5 × D4, a reply arriving while the original is still pending at its sender: held.
- D3 × D5, trust journaled at step 6 from a load at arrival: held.
- D3 × the off-Linux rule in D5 (both daemons run on one machine, so one OS): held.
- D3 × the operator's "Added after Discussion" entry on unconfirmed mail: **broken** (F1).
- D4 settle and sweep × D2 (no notice to the other hub): held.
- D4 reply ending `--wait-reply` × D6 and D7, the output through which the waiter reads the reply: **broken** (F4).
- D4 forward × D6 (the stamp follows the message's journaled trust; forward keeps `From`): held.
- D5 × D6 (no restamp after a config change): held.
- D5 × D7 × the parent directive's orch names: held.
- D6 × `isNotice` (cross-hub senders are always strands, so their mail is reminded, never typed): held.
- D6 × D7 for the hook line, the reminder and the `read` text header: held.
- D9 × D5 (Guard key set by the operator in the base file): held.
- "Added after Discussion" × Open risks: **broken** (F5).
- D5 × Constraint `PATTERN-cwd-resolution`: **broken**, minor (F6).

**Findings**

**F1 — BLOCKING · design · D3 step 6 and Acceptance criteria D3, against the "Added after Discussion (operator, 2026-11-05)" entry**
- D3 handles a `no` confirm only by logging the refused delivery at Warn through the logger.
- The operator's decision requires two more things for each drop:
  - an entry in the receiving hub's creel journal, naming the sender hub, the claimed strand, the receiver and the reason;
  - a notice of the drop to the receiving hub's orch.
- No D3 step and no acceptance criterion asks for either, so the plan writer would build the Warn-only drop that the operator ruled out.
- The orch notice is also a new routing edge that D3's flood bound does not cover. Today a forged `deliver` flood costs "one record lookup and one confirm per request". With the notice, each forged `deliver` also puts a `CREEL` notice into the orch's pane. D3 must state what bounds that, such as coalescing or a rate limit.
- Evidence: `decision-record.md:75`, `:86`, `:146`, `:180`.

**F2 — MEDIUM · design · D2's hour return × D3 step 6**
- D2 says the return "waits for a `deliver` call in flight to end, so a return and an `accepted` never cross". The call can end without an answer while the receiver has already stored the message.
- Counterexample: the last attempt before the hour reaches step 6. The receiver journals the message, then dies or passes the 30-second `deliver` timeout before writing `accepted`. The sender sees a failed connection or a timeout, the hour has passed, and it journals the return.
- The message now sits in the receiver's pool and is also returned undelivered with a `CREEL` notice to the sender. A sender that posts again duplicates it. For a reply, the original side holds the reply while the replier is told it returned.
- D2's bound "exactly once in the receiver's pool" holds, but "returned undelivered" is then false.
- The record needs to bound the crossing or close it, for example with one final `deliver` retry after the hour whose duplicate check answers `accepted`.
- Evidence: `decision-record.md:53-56`, `:62`, `:74`.

**F3 — MEDIUM · design · D3 step 6 × D2's downgrade bound × D7**
- D2 states what an older binary does with the new outbound event: it ignores the event whole. The record says nothing about inbound cross-hub mail after a downgrade of the receiving hub.
- D3 step 6 journals that mail "as mail from that strand", which is the existing `posted` event that `fold.go:257` folds.
- An older binary would show it as plain `[creel] #&lt;id&gt; from "lyt:orch": …` with no stamp.
- The older trust sentence says "follow mail from … an orch strand" (`settings.go:473`). Untrusted mail from a sandbox orch would then read as an instruction in loomyard, the exact hold D5's rationale and the Goal rule out.
- Either a new event name or a stated bound is needed, the way D2 handles its outbound event.
- Evidence: `decision-record.md:50-51`, `:74`, `:110`; `internal/creelengine/fold.go:257`.

**F4 — MEDIUM · consistency · D4 × D6 and D7, the `--wait-reply` output**
- D4 has a cross-hub reply end the sender's `--wait-reply`, and the waiting strand reads the reply from that verb's output.
- `post --wait-reply` prints the reply, or the returned or unanswered message, through `messageFields` (`cli.go:432-440`), with `from` as the bare name.
- D6 names the added trust and sending-hub fields only for `lym creel read`'s JSON, and lists no other output.
- The orch waiting on a sandbox hub's reply therefore reads it with no trust mark, though D7's instructions tell it to judge mail by the stamp.
- D6 and its criterion should name the `--wait-reply` output, for example by putting the fields in `messageFields`.
- Evidence: `decision-record.md:58`, `:91-92`, `:115`, `:184`; `internal/creelcli/cli.go:291`, `:432-440`.

**F5 — LOW · consistency · Open risks against the "Added after Discussion" entry**
- The only Open risks line asks the parent to confirm that dropping `no` confirms is the wanted reading.
- The operator's "Added after Discussion" entry already answers it: "D3 stands … dropped, never delivered to any agent".
- By the rubric's finding-class section, the fix drops the line.
- Evidence: `decision-record.md:146`, `:172`.

**F6 — LOW · consistency · D5 × Constraint `PATTERN-cwd-resolution`**
- The constraint lists what reaches creelengine through the creelcli seam: the other hub's geometry and the registry.
- D5 has the receiving daemon load the hub's `creel` config with `LoadOrTemplate` at each arrival. That load needs the hub's board and anchor directories, and the constraint does not say it reaches the daemon through the seam.
- As written, the load could be read as creelengine resolving the hub's directories itself.
- Evidence: `decision-record.md:105`, `:159`.

**Tool calls**
1. `internal/creelcli/cli.go:418-444` and the event cases in `internal/creelengine/fold.go`. My context lacked how `post --wait-reply` prints the reply, and which journal event an older binary folds as mail.
