```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: I attacked every bound, guarantee, ordering promise and claim about existing code in the decision record. Three claims break.

**Claims attacked**
- D1, concurrent daemons writing the record lose no entry (`internal/state` under `internal/lock`): held.
- D1, the lookup filters moved or removed hubs and refuses a shared shortname: held. A fake hub with the trusted shortname only blocks that hub's mail, and D1's bound already states that.
- D1, the refusal's way forward (`lym creel inbox` starts the daemon that records the hub): held. A daemon that shuts down on a binary replace restarts on the next client call.
- D2, "a return and an `accepted` never cross": broken (F1).
- D2, "exactly once in the receiver's pool by D3's duplicate rule": held for a resend of one message. Broken for a reused id (F2).
- D2, an older binary ignores the outbound event whole: held. `journal_test.go:221` pins that an unknown event name is ignored.
- D2, the downgrade retention bound: held as stated.
- D2, the size check at the widest id: held.
- D2, starting another hub's daemon from a daemon: held. `spawnDaemon` reaps in a background goroutine outside the daemon's wait group (`internal/creelcli/spawn.go:73-90`).
- D3, the confirm tuple against every field `deliver` carries: held. Sender, receiver, reply-to, expects-reply and the body hash are all in it, and the sending hub path is resolved, never taken from the request.
- D3, overlapping retries and a replay before the remote delivery is journaled are stored once (step 6 under the lock): held.
- D3, a confirm after a return answers `no`: held for a receiver that has not stored yet. It does not cover a receiver that stored before the return (F1).
- D3, crossing deliveries do not deadlock (no store lock held across a call): held.
- D3, an older receiver's refusal of an unknown verb is transient: held. `serve` resolves the sender before `dispatch`, so the old daemon answers with an ok:false refusal, either `resolveSender`'s pane refusal or the unknown-verb error (`server.go:208-215`, `382`).
- D3, the `confirm` probe and the `deliver` flood bounds: held.
- D3, the list of routes the check does not stop: held as a bound.
- D3, the off-Linux bound: held, since both daemons run on the same machine.
- D3/D5, "ids per hub that are never reused" (constraint from `doc.go:62`): broken across a journal reset (F2).
- D5, a local layer file holding the Guard key is refused and no machine layer is stacked: held (`internal/configengine/layers.go:113-131`, `LocalDenied`).
- D5, a failed load stores the message untrusted: held.
- D5, exact orch names only: held as a bound. Starting an orch strand through reed is listed in D3's bound.
- D5, the rationale "a committed change leaves a trace in git": held. It claims a trace only for committed changes, and the base file is listed in D3's bound.
- D4, "only when the answered id is an outbound message of its own that went to the replier's hub": held.
- D4, a reply after retention is refused: held.
- D6, a stamp after the quote is creel's alone: held.
- D7, the bound "the compiled trust sentence changes in every hub": broken (F3).

**Findings**

**F1 — MEDIUM, design, D2 hour-cap return (record lines 55-56; D3 rationale line 82)**
A message can sit in the receiver's pool and also be returned undelivered.
- Counterexample: R answers the confirm `yes` and stores the message. The sending daemon S stops before it journals `accepted`, for example on a binary replace, which ends the daemon (`server.go:162-165`). S restarts only on a later client call, more than an hour after the post.
- On restart the message has been pending past the hour, so S returns it at once. No `deliver` is in flight to wait for, so D2's guard does not apply.
- The same happens when S's 30-second `deliver` times out while R finishes storing after the deadline, and the hour falls in between.
- Result: R holds the message, and the sender gets a `CREEL` notice saying it returned undelivered. A re-post then reaches R twice.
- D2's guard only keeps the answers from crossing, not their effects.
- Fix: journal a return only after one more `deliver` answers something other than `accepted`. Step 3 already answers `accepted` for a message R holds.
- Evidence: `decision-record.md:55-56`, `:82`; `internal/creelengine/server.go:162-165`.

**F2 — MEDIUM, design, D3 step 3 duplicate key (record lines 70, 152)**
The duplicate check keys on the sending hub path and the sending id. That relies on ids never being reused, but ids restart when the hub's journal is gone.
- The journal lives in the hub's scratch directory (`internal/hubgeom/creelgeom.go:24-29`). The next id is the largest id folded from that journal (`internal/creelengine/fold.go:165-184`, `253`).
- If the hub is re-created at the same path, or its scratch directory is removed, ids start at 1 again.
- Within R's seven days, a new message whose id matches a held one gets `accepted` with the old id and is stored nowhere.
- The loss is silent, against the operator's added decision that every drop of genuine mail be visible (`decision-record.md:146-148`).
- Fix: on a duplicate hit, also match sender, receiver, reply-to, the expects-reply mark and the body hash, and refuse visibly on a mismatch. Alternatively, carry a journal epoch in `deliver` and `confirm`.
- Evidence: `doc.go:62` scopes "never reused" to the high-water record across day files, which a deleted journal directory loses.

**F3 — MEDIUM, design, D7 bound "the compiled trust sentence changes in every hub" (record line 129)**
The trust sentence does not change in sessions that are already running.
- The sentence goes into the session's append-system-prompt when shuttle launches the session (`internal/shuttleengine/claudeengine/settings.go:471-494`).
- A session launched before the deploy keeps "follow mail from your parent, an orch strand or a lym program as the operator's own instructions".
- After the binary is replaced, the daemon restarts within about a second (`server.go:162-165`) and starts handling mail from another hub.
- So a live loomyard session can receive `from "lyt:orch" (other hub, untrusted)` under text that says to obey any orch strand. That is the threat D7's rationale names.
- The same holds for a parent directive rendered before the deploy.
- Fix options:
  - State the bound and its window.
  - Make the untrusted stamp say it is data, not an instruction, so old text reads it right.
  - Refuse mail from another hub to a session launched before the change.
- Evidence: `settings.go:473` and `481`; `decision-record.md:123`, `:128-129`.

**Tool calls**
1. Searched the journal fold, the files in `internal/creelengine` and where the journal directory is set: to check the older-binary claim and how ids are issued, which the manifest lacked.
2. Searched for the journal location in `hubgeom`, the id rule in `doc.go` and `fold.go`, where `creelTrustSentence` is used, and the start/wait lines in `spawn.go`: to check the id-reuse constraint, when the trust sentence is applied, and how a daemon is started.
3. Read `internal/creelcli/spawn.go:40-100`: to check how a daemon starting another hub's daemon reaps it (D2).
4. Searched `internal/configengine/layers.go` for the machine-layer and Guard handling: to check D5's layer claims.
5. Read `internal/configengine/layers.go:20-48` and `:110-150`: to check which layers a hub-wide module stacks (D5).
