```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge each reviewer proposal the fixer carried out as written, by building a case where the proposal is wrong or breaks something elsewhere in the record.

Proposals challenged: F1, F2, F3, F4, F5, F7, F8, F9, F10, F11, F12, F13, F14, F15. F6 is a decision finding and was not fixed.

**MEDIUM: F3 narrows the race but the new rationale says it is closed**
- Changed text: D3 step 5 ("still pending, with no remote delivery and no return journaled") and the D3 Rationale ("and a message the sender was told failed is never stored").
- The case:
  1. A `deliver` is in flight close to the hour cap.
  2. The receiver's `confirm` reaches the sender while the message is still pending, so the sender answers `yes`.
  3. The sender's hour cap fires and journals a return. It also sends the `CREEL` notice and ends `--wait-reply` as returned.
  4. The receiver gets its `yes`, stores the message under step 6 and answers `accepted`.
- The message is stored and acted on after the sender was told it failed. The proposal only shrinks the window from "after a return" to "between the confirm and the return".
- D2 also never says what the sender does with an `accepted` that arrives after a return is journaled. It does not say whether the sender journals a remote delivery or ignores the answer. The case therefore leaves the sender's journal holding a "returned" message that was in fact delivered.
- Evidence: D2 says "`accepted` is journaled as the message's remote delivery". The return rules say the message "returns undelivered" at the hour. No rule orders the return against a `deliver` in flight. The rationale sentence states the guarantee outright.
- A fix would make one of these changes:
  - Hold the return until the in-flight `deliver` call ends, which is at most 30 s.
  - Or narrow the rationale to "never stored once the return is journaled before the confirm".

**LOW: F13 drops the broker-route fact, and nothing else in the record keeps it**
- Changed text: the second Open risks bullet, now removed.
- The removed bullet carried a fact D3's bound never states: a sandbox hub inside a closed area could not reach loomyard's socket, so cross-hub mail would need a route through the area's broker.
- D3's bound only hands the forging routes to `sandbox-closed-area-build`. It does not say the closed area breaks this feature's transport. A reader of the note would not learn from this record that cross-hub mail depends on reaching the other hub's socket directly.
- The support log's turn-3 line also still says this exact risk was "rejected: the risk carries the broker route… that the bound does not raise", which makes that rejection reasoning stale. The new Round 1 entry gives the review's counter-reason, so the log reads in order. The record itself, though, now holds no trace of the dependency.
- Evidence: the support log's turn-3 leanness line, and D3's last forging Bound.

**LOW: F12's flood cost is stated as exact, but steps 1 to 3 can answer without the confirm**
- Changed text: the new D3 Bound ("a `deliver` flood costs the receiver one record lookup and one confirm per request").
- The case: a flood of requests that are malformed, from the receiver's own shortname, or duplicates of a held id is answered at step 1 or step 3. Those requests cost no confirm, and step 1 refusals cost no lookup.
- What the bound should give is an upper limit ("at most"). The stated per-request cost is wrong for those requests. The exposure claim itself is unaffected.
- Evidence: D3 steps 1 and 3 answer before step 4.

The other proposals hold up:
- **F9:** An entry kept while its record is unreadable is still filtered at lookup.
- **F8:** A held id answers before the receiver rules apply, which is correct for a message already stored.
- **F7:** A reply that arrives after the seven-day carry ends is already bounded in D4.
- **F2:** The re-check under the lock holds no lock across a call, so it agrees with the lock bullet.
- **F1:** The `refused` answer now needs a form that an older binary's error reply cannot take. The record implies this, and it can be implemented.
- **F4, F5, F10, F11, F14, F15:** I found no case against them.

Further tool calls: none.
