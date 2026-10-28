```yaml
lens: fix-proposal
outcome: reported
```

Lens fix-proposal: I challenged the reviewer proposals the fixer carried out as proposed, which were all findings except F2.

Proposals challenged: F1, F3, F4, F5, F6, F7, F8, F9, F10, F11, F12, F13, F14, F15, F16.

1. **MEDIUM:** D10 now says a reply to a chase from anyone other than the holder "is refused, naming the way forward: post to the original sender by name."
   - That way forward fails for exactly the senders the review flagged.
   - When the original sender is a lym program, the post lands in a mailbox nobody reads, which is the defect F5 describes. The review says `<shortname>:CREEL` "never reads". D10's own text expects a program sender to have possibly never read ("a program that has read once (otherwise the journal record alone stands)").
   - When the original sender is EXTERNAL, a strand cannot post to it by name. `Store.Reply` (`internal/creelengine/verbs.go:202-208`) reaches an EXTERNAL sender only by answering a waiting call; it never delivers mail to one.
   - The forwardee also does not hold the chased message, so its post would never clear the mark.
   - Concrete case: `lym loom decision add` posts as LOOM with `--expect-reply`. The holder forwards the chase. The forwardee replies to the chase, is refused, and is told to post to LOOM, which never reads.
   - The reviewer's other option, that chases cannot be forwarded, avoids this. So does a refusal that says only the holder of the chased message can answer it.

2. **LOW:** D10 now says "The chase number and the chased message's id travel on the chase's own `posted` record, which the restart counts, so a crash between the chase's post and its `chased` record never yields a third chase."
   - A crash in that window still leaves no `chased` record. Nothing says the restart writes it.
   - D11 says the journal gets "a `chased` event per chase", and the D11 acceptance row says `lym creel log --id` lists the `chased` events. Both fail for that chase.
   - Fix: state that the restart writes a missing `chased` record from the chase's `posted` record, or make the `posted` record serve as the `chased` event.

3. **LOW:** D3 now says "The burler, loom and webster wirings that call `orchcli.NotifyParent` unconditionally change to that rule."
   - Read literally, this also covers loom calls that are not runner notifiers:
     - The hand-back `Notify` at `internal/loomcli/drivewiring.go:243`, which is paired with `OrchRecorded: orchcli.ParentOrchRecorded(location)` on the line below.
     - The landing `Notify` at `internal/loomcli/landingdeps.go:204`.
     - The discussion `NotifyParent` at `internal/loomcli/wiring.go:587`.
   - A plan writer who switches the hand-back `Notify` to `NotifyFrom` for a beam location, but not `ParentOrchRecorded`, makes the recorded check look at one orch while the post goes to another.
   - Fix: limit the sentence to the runners' `SetNotifier` wirings (`loomcli/wiring.go:465`, `burlercli/wiring.go:153`, `webstercli/wiring.go:212`), or name the `OrchRecorded` twin as changing too.

Tool calls:
- One `sed` over `internal/loomcli/landingdeps.go`, `internal/loomcli/wiring.go`, `internal/loomcli/drivewiring.go` and `internal/burlercli/wiring.go` around their `NotifyParent` calls. My context had those calls only as grep hits, so it could not show which ones are runner notifiers and which carry an `OrchRecorded` twin.
