```yaml
lens: bound-breaker
outcome: reported
```

Task: run the bound-breaker lens on the slack-v2 decision record by attacking each of its claims and bounds with a counterexample.

**Claims attacked**
- D1 "at most one relay per machine holds a Socket Mode connection": held. Only the lock holder connects. Login's check connection is a short-lived non-relay connection, and the record covers the retry it causes.
- D1 "the lock wait blocks without waking": held. `lock.AcquireWriteLock` blocks on flock (`internal/lock/lock.go:30-40`).
- D1 "an envelope is acknowledged only after the hint write stood; a failed or slow write owes a resync": held.
- D1 "a planted hint can at most make a relay read its own channel": held.
- D1/D3 "a map that cannot be read fails closed": broken when the live-hub list itself cannot be read (F3).
- D2 "the inbox directory holds only hints; any other entry is removed unread": broken by an atomic hint write (F4).
- D2 "cursors only move forward; two wakes never read one channel at once": held for reads. Ensures are not serialized (F6).
- D2 thread floor and prune bounds: held as stated.
- D3 "an older binary still reads the bot token under `token`": held.
- D3 "a relay missing a token tells the operator through the prime's channel": broken on the upgrade path (F2).
- D3 "login writes a token only when it matches the stored tokens": undefined when no bot token is stored yet (F8).
- D3 user-token bound: held for in-map channels. It passes vacuously when the hub list fails (F3).
- D4 "`prime_beam` is a name no slug can produce": held.
- D4 "names can repeat only across machines": held. A shortname has no `-` (`internal/agentname/agentname.go:86-95`), so the `&lt;shortname&gt;-&lt;machine&gt;-` prefix is unique on one machine.
- D5 "the map is the authority; on `name_taken` the list settles it": broken for a channel the map does not hold (F1, F7).
- D5 duplicate-shortname race bound: held as stated. A same-relay race exists too (F6).
- D6 "archiving after the relay has ended means no live relay reopens it": held. A later reopen is the designed behaviour.
- D8 "`refusalCap` in `internal/slackengine/outbound.go`": held (`outbound.go:32`).
- D8 one count per refused post: held.
- D10 "an old `setup.json` needs no migration": held, since `state.WriteJSON` keeps unknown keys (`internal/state/state.go:27-29`).

**Findings**

**F1. MEDIUM · design · D5 step 2 and D6. Reopening an archived channel the map does not hold leaves its cursors undefined.**
- D6 says "the cursors stay where the entry left them" (`decision-record.md:263`). On D5's `name_taken` path there is no entry, which happens after:
  - a re-clone, since `.lym` is never tracked so `channels.json` is gone;
  - D3's own way forward of moving the map aside (`decision-record.md:172-173`).
- The only defined start for an entry without cursors is D5 step 1's empty cursor (`decision-record.md:225`). That reads from the channel's beginning and forwards the archived beam channel's whole operator history to the orch again.
- The adopt branch defines its start, at the newest message (`decision-record.md:230`). The reopen branch does not (`decision-record.md:229`).

**F2. MEDIUM · design · D3. A relay that re-execs onto this task's binary while the operator is away tells the operator nothing.**
- Before the upgrade no `channels.json` exists. So "when … the map holds the prime's channel, it posts the same line there" (`decision-record.md:160-162`) never fires on the path the record names, a re-exec onto a binary of this task.
- After that:
  - The notice goes to the orch alone.
  - Presence stays `slack`.
  - Dobby's `EnsureRelayWhileAway` answers `OutcomeNotConfigured` quietly (`internal/beamcli/dobbydeps.go:361`, `internal/slackengine/start.go:180-181`).
  - The one channel the operator watches, the old `setup.json` channel, is one "nothing reads" (`decision-record.md:329`).
- The acceptance criterion covers only a map that already holds the prime's channel (`decision-record.md:436-437`).

**F3. MEDIUM · design · D1 and D3. A failed read of the live-hub list has no stated bound.**
- `hubregistry.Live` returns an error for a `hubs.json` it cannot read or decode (`internal/hubregistry/hubregistry.go:98-102, 146-149`).
- The record fails closed only for "a live hub's map that cannot be read" (`decision-record.md:48-49, 171-172`).
- If the holder takes a failed list as zero hubs, two things pass vacuously:
  - "every live map reads", so every event is acknowledged and dropped;
  - the user-token test "held by no other orch's entry in any live hub's map" (`decision-record.md:166`).
- Nothing says the failed list leaves events unacknowledged and refuses unarchive, invite and adopt.

**F4. MEDIUM · design · D2. "The inbox directory, which holds only hints" combined with "removes any other hint unread" races with an atomic hint write.**
- `internal/state` writes through a temp file and a rename (`internal/state/state.go:30-41`, ending in `fsx.AtomicWriteBytes`). Its lock goes in a `locks/` directory beside the record (`internal/slackengine/geometry.go:65-67`).
- A hint written that way creates a temp entry and a `locks` directory in the inbox. The temp file's create event wakes the relay, which finds a name that does not parse and removes it.
- The holder's rename then fails, the write counts as failed, and the hint waits for the owed resync at the next 30-second ping. Push degrades from about a second to up to 30 seconds whenever the race hits.
- The record says neither how a hint is written nor that removal spares a write still in progress.

**F6. LOW · design · D5. Two ensures of one beam channel inside one relay are not serialized.**
- The outbound goroutine (first routed mail) and the drain (presence `slack`, every listed beam) can both call `conversations.create` (`decision-record.md:219-221`).
- The loser gets `name_taken`. The channel passes the map test, because its holder is the same orch, so it is adopted with its cursor at the newest message (`decision-record.md:227-230`).
- Any operator message written between the winner's invite and the adoption is skipped, and the operator is invited twice.
- D2's single-goroutine rule covers reads only (`decision-record.md:113`).

**F7. LOW · design · D5 step 2. A live channel this bot created and no map holds, with the bot not a member, falls into no branch.**
- The branch admits only an archived channel or a live one the bot is a member of (`decision-record.md:227-230`).
- "Any other is refused" (`decision-record.md:231`) is the sibling of the creator test, not of the member test.
- This happens when the operator removes the bot from a channel, so the outcome is unstated.

**F8. LOW · design · D3. The first login of an `xapp-` or `xoxp-` token on a fresh machine has no stated outcome.**
- "An app-level or user token that does not match the stored bot token is refused" (`decision-record.md:150-151`) assumes a stored bot token.
- With none stored, the record says neither "refuse, naming login of the bot token first" nor "accept".

**Tool calls** (5 used):
1. grep of `internal/agentname` and `internal/lock`: the shortname grammar and the lock API were not in the manifest.
2. Read of `lock.go:25-45` and `agentname.go:80-125`: to confirm the blocking acquire and the shortname grammar's lack of `-`.
3. grep of `internal/state` and the repo for temp-file writes: to learn how an atomic write lays out its files, which the manifest lacked.
4. grep across `internal/` for `CreateTemp` beside a file: same reason, to see whether temp files sit in the target directory.
5. Read of `internal/state/state.go:25-60`: to confirm `WriteJSON` writes atomically and keeps unknown keys.
