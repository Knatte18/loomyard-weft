```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced the code paths that already read or write the state D1–D9 add or change.

**Findings**

1. **BLOCKING — D2 and D4: creel's removed-holder sweep would settle every pending or delivered outbound message, and the record does not say otherwise.**
   - D2 says only that a message to another hub "is journaled as an outbound message with the receiver's full name", and that retention carries it "as it carries unread mail".
   - The sweep in `internal/creelengine/sweep.go` (`sweepNeeded`, `sweepOnce`, `isStrandName`) counts any holder that parses as a strand name and is missing from two successful listings as gone. That includes a foreign `lyt:orch`, which never appears in the sending hub's own strand list.
   - The one exemption, `waitsForHubOrch`, covers only program mail to an orch of the same hub.
   - If the outbound message keeps the receiver as its holder:
     - An undelivered one is sent one step back to its sender, or journaled undelivered, within about 60 seconds. That defeats D2's hour of retries.
     - A delivered one that expects a reply has its holder read as gone and is settled unanswered with reason `holder-gone`. That sends the sender a `CREEL` notice and ends its `--wait-reply` with `unanswered` before D4's reply can arrive.
   - Fix: D2 should say that an outbound message is held by no local strand, or is otherwise exempt from the sweep, chase and settle until it is delivered or returned. It should also say what holds it once delivered, so D4's `--wait-reply` survives.

2. **MEDIUM — D2: the per-receiver delivery worker and the mail flags would treat an outbound message as local unread mail.**
   - Today `internal/creelengine/flags.go` wants a flag for every unread message's `Holder`, so a foreign name would get a flag file in the sending hub's FlagDir. The daemon's reconcile at start would recreate it.
   - The Scheduler starts one worker per receiver with unread mail. In `deliver.go` that worker's `findStrand` finds no foreign strand, so it probes and backs off for nothing alongside D2's separate outbound worker.
   - Fix: D2 should keep outbound messages out of the receiver-keyed pool paths (flags, delivery workers, unread reminders), as it does for chases.

3. **MEDIUM — D2 and D6: the receiver measures the 8 KiB line cap on the stamped form, but the sender checks a post before the stamp exists.**
   - D6 says the cap measures the stamped line, which only the receiving daemon renders.
   - The sending daemon checks the size at the post. A body that fits the plain line but not the stamped one is accepted with an id, then refused at delivery and returned.
   - Fix: D2 should have the sending daemon measure the longest stamped form when it accepts a post to another hub.

4. **LOW — D2 and the tolerant-records constraint: an older binary would misread a newer journal.**
   - An older daemon folding a journal that holds outbound records ignores the new events. It still folds the outbound post itself as ordinary mail held by a foreign strand name, and its sweep then returns that mail.
   - The window is short, since a replaced binary ends the daemon within a second. The Constraints line claiming that ignoring unknown events is enough overstates it, though.

5. **LOW — D7: a hub whose board copy of the parent stencil was edited by hand keeps the old trust text.**
   - `PATTERN-stencil-ownership` says stencilstore never overwrites a hash-mismatched stencil. In such a hub, `parent-directive-parent.md` keeps counting any orch strand as trusted, even once the binary carries D7.
   - The Claude trust sentence is compiled in, so it does change. The record is silent on the stencil case.

**Paths traced**
- D1:
  - The daemon re-exec after a replaced binary records the hub again: survives.
  - lymtemp's `Root.Sweep` does not touch the record, which lives in the state dir: survives.
  - A clone through `fabricengine.CloneAndWire` writes nothing until the hub's first daemon start: survives, and the record states it as a bound.
- D2:
  - Removed-holder sweep: breaks (finding 1).
  - Chase of the outbound copy: survives, since it chases only messages a strand holder has read.
  - Flags and their reconcile at start: break (finding 2).
  - Per-receiver delivery worker: breaks (finding 2).
  - Retention carry: survives as stated.
  - Replaced-binary shutdown: survives. Deliveries in flight finish or fail and are retried, and a confirm sent to a closing listener fails and is retried.
  - Older-binary fold: breaks briefly (finding 4).
  - Sender's size check: breaks (finding 3).
- D3:
  - The client refuses a response whose hub path differs: survives, since it is built from the geometry of the hub the registry names.
  - `checkProgramClaim` refuses another hub's program: survives, consistent with D2.
  - Receiver's pause or hold: survives, since a post still stores and delivery waits.
- D4:
  - Reply verb, where only the holder may reply: survives, given the outbound branch.
  - Sweep of a gone holder on the receiving side: survives as the record states.
  - Sending side `--wait-reply`: breaks through finding 1.
- D5:
  - Config load from the hub directory with `Dirs{Board}`: survives.
  - `hubreconcile` and the sparse base: survive, since the module is new.
- D6:
  - Hook claim, reminder and `lym creel read` rendering: survive as stated.
  - Size check: breaks (finding 3).
- D7:
  - Compiled trust sentence: survives.
  - Board stencil copy: breaks when hand-edited (finding 5).
  - Orch role, rendered from embedded bytes: survives.
- D8:
  - The privilege checks for inject, interrupt and hold: survive with the new refusal in front of them.

**Tool calls**
- Grep of `internal/creelengine/sweep.go` for how a holder is judged gone and the `waitsForHubOrch` exemption: my context held only the doc's summary of the sweep, not its holder test.
- Grep of `internal/creelengine/deliver.go` and `flags.go` for how the worker and the flags key on the holder: my context did not show whether those paths key on any holder name.
