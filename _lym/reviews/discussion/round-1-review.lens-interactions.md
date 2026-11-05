```yaml
lens: interactions
outcome: reported
```

Task: the interactions lens. I checked each pair of decisions that act on the same state and each acceptance criterion against the decisions it covers.

**Pairs checked**
- D1×D2 (post resolves the receiving hub through the record): held.
- D1×D3 (receiver resolves the sender's hub through the record; refuses on none or several): held.
- D1×D2 (the outbound worker starts another hub's daemon, which records itself at start): held.
- D1 start-time pruning × concurrent lookups and records under `internal/lock`: held.
- D2 30 s `deliver` timeout and retry × D3 step 3 duplicate rule, when the `accepted` answer is lost: held for sequential retries.
- D3 step 3 duplicate check × D3 lock rule × D3 step 6 store, with two deliveries of one tuple in flight at once: **broken** (F1).
- D2 one-hour or `refused` return × D3 step 5 confirm rule: **broken** (F2).
- D2 retention × D4 settle with no notice to the other hub's sender: **broken** (F3).
- D2 retention × D3 confirm after retention dropped the message (answers `no`): held.
- D2 retention × D4 late reply after retention: held, bounded in D4.
- D2 returned reply × D4 replier-side replied mark: held, bounded in D4.
- D2 strand-only receiver × D4 forward refusal (program or other hub): held.
- D3 step 1 own-shortname refusal × D4 replies crossing back: held.
- D3 step 3 duplicate rule × D4 replies (same key of sending hub path and id): held.
- D3 confirm starts no daemon × D2 worker starting the receiver: held.
- D5 trust at arrival × D3 step 6 journaling × D6 stamp: held.
- D5 Linux-only trust × D3 off-Linux bound: held.
- D5 exact orch names × D7 parent directive's "orch strand of this hub (role `orch`)": held.
- D5 × D9 (`trusted_hubs` in lyt and cs, set by the operator in the Guard base file): held.
- D6 stamped cap × D2 size check (longer stamp, widest id) × D3 step 1 refusal: held.
- D6 "agent mail, reminded not typed" × D2 `CREEL` notice (a local program): held.
- D8 refusals × D2 post route, `post --parent` unchanged: held.
- D8 `hubs` × D1 check and D5 trust: held.
- Auto-mode assumptions × D1, D2, D3, D5, D8: held.
- Constraints (daemon-wakeups, child-processes, cwd-resolution, test-isolation) × D1, D2: held.
- Acceptance criteria × D4 and D6: **partly uncovered** (F4).

**Findings**

F1. MEDIUM, design. Location: D3 steps 3, 4 and 6, the lock paragraph, and the second bound.
- **Summary:** the duplicate check runs before the confirm and outside the store lock, so two deliveries of one tuple in flight together both pass step 3 and are stored twice. This breaks D3's bound "a replay of the same tuple is stored once by step 3" and D2's "exactly once in the receiver's pool".
- **Concrete case:** a hand-written replay sent while the real delivery waits on its confirm. Both get `yes` and both store.
- **Fix:** the record needs step 6 to repeat the duplicate check under the store lock before journaling.
- **Evidence:** `_lym/discussion/decision-record.md:68-78`, and `internal/creelengine/doc.go:73` (the pool holds each message once).

F2. LOW, design. Location: D2 return rule × D3 step 5.
- **Summary:** a message D2 returned undelivered, after a `refused` answer or the hour, has no remote delivery journaled. So by D3 step 5 the sending daemon still answers `yes` to its confirm.
- **Concrete case:** a `deliver` of that tuple arriving later is stored and reaches the agent, though the sender was told it never arrived and may have posted again.
- **Fix:** the record needs the confirm rule to answer `no` for a returned message as well.
- **Evidence:** `_lym/discussion/decision-record.md:53-55` and `decision-record.md:70`.

F3. LOW, design. Location: D2 retention × D4 settle.
- **Summary:** D2 retention carries a delivered outbound message while it expects a reply and has none. D4 settles such a message unanswered on the receiving side and sends the other hub no notice, so the sending side carries it at every retention, with no end.
- **Same gap:** a returned message that expects a reply also matches D2's carry rule.
- **Fix:** the record needs an end to the carry, such as the hour cap or a settle notice across hubs.
- **Evidence:** `_lym/discussion/decision-record.md:57`, `decision-record.md:90` and `decision-record.md:93`, and `internal/creelengine/doc.go:68`.

F4. LOW, consistency. Location: the Acceptance criteria for D4 and D6.
- **Summary:** some behaviour the decisions state has no criterion.
  - D4's replier-side replied mark, which stops the chase.
  - D4's settle that sends no notice to the other hub.
  - D6's `lym creel log`, showing the sending hub's path and id.
  - D6's reminder stamp.
- **Evidence:** `_lym/discussion/decision-record.md:88-90` and `decision-record.md:109`, against `decision-record.md:171` and `decision-record.md:173`.

Tool calls: none.
