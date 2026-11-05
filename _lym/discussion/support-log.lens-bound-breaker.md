```yaml
lens: bound-breaker
outcome: reported
```

**bound-breaker report: creel-cross-hub-mail decision record**

**Findings**

1. **MEDIUM — D3 (steps 2, 5 and 6) and D2's "exactly once in the receiver's pool": a crash loses a legitimate message.**
   - The sending daemon journals `confirmed` before it answers `yes`.
   - Suppose the receiving daemon crashes, or its journal write fails, after the `yes` and before it stores the message. The sender sees a failed call and retries.
   - On the retry, step 2 finds no stored message, so step 4 confirms again. The sending daemon now answers `no`, since the id was already confirmed, and step 6 refuses.
   - The sender takes `refused` as permanent, so the legitimate message returns undelivered.
   - The record says "nothing legitimate is lost by refusing", and that does not hold here.
   - Fix direction: one of these.
     - Let a repeat confirm of the same tuple answer `yes` while the outbound message has no journaled remote delivery.
     - Have the receiver journal a pending arrival before it confirms.

2. **MEDIUM — D3 does not say whether the receiving daemon holds its store lock while it calls confirm, so two hubs can deadlock.**
   - Connections are served concurrently: `server.go` spawns a goroutine per accepted connection.
   - D2 forbids holding the lock only in the outbound worker.
   - Counterexample: `ly:orch` posts to `lyt:orch` while `lyt:orch` replies to an earlier `ly` message.
     - Each daemon's `deliver` handler is inside its own store lock and calling confirm on the other.
     - Each confirm handler then waits for the lock the other side's `deliver` handler holds.
   - Fix: state that the receiving handler holds no store lock across the confirm call.

3. **MEDIUM — D3's bound treats creel's verbs as the attack path, but a sandbox agent can get a trusted orch name through reed.**
   - D3 says the check stops a sender forged through creel's verbs. The trust it guards (D5) rests on the sender's *name* being an orch of a trusted hub.
   - Counterexample: an agent in `lyt` runs reed's verbs (`lym reed add` or a window verb) from loomyard's hub directory to create a strand whose role is `orch`, then posts from that pane. Creel resolves it as `ly:<slug>:orch`, the sending daemon confirms it, and `lyt` stamps it trusted.
   - Another unlisted route: typing into `ly:orch`'s pane through its tmux socket, an exposure `sandbox-closed-area-build` names.
   - Fix: add both routes to the bound, and decide whether a reed-added strand may take the `orch` role.

4. **MEDIUM — D1: the record is never pruned, and tests would write to the operator's real file.**
   - The lookup drops dead entries but no writer removes them.
   - Every `hubforge` test hub (`TestShortname = "tst"`) starts creel daemons. I found no `HOME` or `XDG_STATE_HOME` isolation in the testkits, so each test run appends `tst` entries to the operator's `~/.local/state/lym/hubs.json` and they stay.
   - Two live test hubs in one test both carry `tst`, so a cross-hub test between them hits D1's ambiguity refusal.
   - `PATTERN-test-isolation` requires hermetic tests.
   - Fix: a writer that prunes, the state root injected so tests can redirect it, and distinct shortnames in two-hub tests.

5. **LOW — D3 steps 2 and 3 are out of order.**
   - Step 2 keys its duplicate check on "the sending hub path", but only step 3 resolves that path, and step 3 says never to use the path the request names.
   - As ordered, step 2 can only use the request's path.
   - Fix: resolve through D1 first, then check for a duplicate.

6. **LOW — D5's "an unreadable file trusts nobody" misstates what `LoadOrTemplate` does.**
   - Per `PATTERN-config-layers`, `Load` and `LoadOrTemplate` differ only on an absent base file. A malformed file is an error, not a template fill.
   - Fix: say that a load error stores the message untrusted.

7. **LOW — D2's one-hour cap does not survive a restart unless it counts from a journaled time.**
   - The daemon exits on every binary replacement, which happens at every deploy.
   - If the hour is counted in memory from the first attempt, a restart resets it, and the cap is unbounded across deploys.
   - Fix: count from the post's journaled time.

8. **LOW — D4 against D2: a program can hold mail it cannot answer.**
   - D4 lets "its current holder" reply. D2 lets only a strand send across hubs.
   - A program receiver such as `lyt:SLACK` from `reed-slack-relay` can hold a cross-hub message but cannot answer it.
   - Fix: refuse program receivers for cross-hub mail, or state the reply refusal.

9. **LOW — D2 retention against D4: a late reply can be refused.**
   - After delivery, the sending side's outbound message has no local holder, so it is neither unread nor chaseable, and retention stops carrying it after 7 days.
   - A reply that comes later is refused under D4, because the answered id is gone, and so is a re-confirm.
   - Fix: state the bound, or have retention carry an outbound message that expects a reply until it is replied.

**Claims attacked**
- D6 "a stamp after the quote is creel's alone": held. `MessageLine` renders only `msg.From`, never the EXTERNAL envelope, and control characters in the body collapse into one line after the colon.
- D2 "serves confirm on another connection": held for the worker, since connections are served by a goroutine each. Broken for the receiving handler (finding 2).
- D3 replay "confirm yes once": held, but it causes the loss in finding 1.
- D3 "never uses a hub path the request names": held in step 3, contradicted in step 2 (finding 5).
- D3 off-Linux untrusted: held, since both daemons share the machine's OS.
- D1 lookup re-checks the Bolt shortname: held. Pruning and test isolation are broken (finding 4).
- D1 "every hub that ran a creel verb is in it": held, since a replaced binary restarts the daemon on the next verb.
- D4 reply only to own outbound for replier's hub: held, since a forged reply still has to pass the replier hub's confirm.
- D5 Guard keeps the key out of the local layer: held per `PATTERN-config-layers`.
- D5 "unreadable trusts nobody": broken in wording (finding 6).
- D2 hour cap: broken across restarts unless journaled (finding 7).
- D4 "by its current holder": conflicts with D2 for a program holder (finding 8).

**Tool calls**
- Grepped `server.go`, `pool.go` and `layers.go` for the accept loop, the receiver check and `CategoryGuard`; the context lacked the daemon's concurrency model.
- Grepped `server.go` for `AcceptUnix` and `go func`, `checkReceiverLocked` in `pool.go`, and `fabriccli/shortname.go`, to confirm a goroutine per connection and the receiver rules.
- Read `MessageLine` in `reminder.go`, to check whether the EXTERNAL envelope reaches the line (D6 claim).
- Grepped for `hubforge` shortname files and `XDG_` in the testkits; the context lacked the test hub shortname and HOME isolation.
- Grepped `hubforge/hub.go` for the shortname and testkit `HOME`, giving `TestShortname = "tst"` and no isolation found.
