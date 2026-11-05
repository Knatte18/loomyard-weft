```yaml
lens: board-scope
outcome: reported
```

Lens board-scope: checking the decision record against the board entry, its touching entries and its peers. I made no further tool calls, because the context already held the task entry, the board list, `lym loom peers`, the `reed-slack-relay` decision record and the touching notes.

**Findings**

1. **LOW, D1 (machine record).** The entry says the shortname-to-path record is "written when a hub is created or opened". D1 writes it only when the hub's creel daemon starts, so a hub that has never run a creel verb cannot be addressed.
   - Evidence: the entry's Proposals; D1's bound; the support log rejects registering at clone because of fabric's boundary.
   - The departure is reasoned and the starting points were "not final", but the record never says it departs from the entry. D1's rationale needs one line saying so.

2. **LOW, D2/D5 against peer `reed-slack-relay`.** That peer's D5 posts to Slack only mail "whose stamped sender is an orch strand or a program of the hub". Under this record's D2, `ly:SLACK` is a valid receiver for a strand of another hub, because a program that has read is a receiver.
   - If the relay's check reads only the role from the name, mail from `lyt:orch` to `ly:SLACK` would reach the operator's Slack as if it came from an orch of this hub.
   - Fix in this record: either refuse cross-hub posts to a program receiver, or state that a receiving filter must check the sender's shortname as well as its role.
   - Evidence: this record's D2 and D8; the `reed-slack-relay` record's D5 third bullet and its D7 bound on program trust.

3. **No finding, Open risks (unverified mail).** The entry says mail that cannot be confirmed "stays unverified" and shows as unverified. D3 refuses it instead, and a sending daemon that is down is not started for the confirm.
   - The record already raises this under Open risks for the parent, which is the right channel. I am not raising it again as a `decision` finding.

4. **No finding, D9 (trial).** The entry asks for a darn `docs` task. Filing a note follows the rule that a run never creates a task. The operator promotes it.

**Goal points of the entry**

| Goal point | Status |
|---|---|
| Orchs of different hubs mail each other | Met (D2) |
| The receiver sees the real sending strand, not `EXTERNAL` | Met (D3, D6) |
| Receiver derives the sending hub's socket from its hub path | Met (D1, D3 via `CreelGeometry`) |
| Confirm call carries the id, receiver and text hash, and answers yes or no, never the text | Met (D3) |
| A message is confirmed once, so a replay is refused | Met (D3) |
| Confirmed on arrival, within retention | Met (D3, D2 retention) |
| A sending daemon that is down is started, or the message stays unverified | Deviates; raised under Open risks (finding 3) |
| Works within one user only | Met (Constraints, D3 bound) |
| Direct transport, with the inbox folder weighed | Met (D2; rejected in the support log) |
| Addressing by shortname through a machine record | Met, but written at a different moment (finding 1) |
| Replies cross the hub border | Met (D4) |
| `trusted_hubs` in the hub's config | Met (D5) |
| A trusted orch has authority; other strands, unlisted hubs and unverified mail are data | Met (D5, D6; unverified mail never reaches an agent) |
| One-way trust: `lyt` and `cs` list `ly`, loomyard lists none | Met (D5 default empty, D9 setup) |
| Roles and the shuttle hooks' creel note learn the new sender kinds | Met (D7) |
| Not in this round: another machine, the closed area | Respected (Scope Out) |
| Trial at the end | Met as a note (D9) |
| The "seen again" case: steering `lyt`'s reviewer and `lyt:orch` | Met through the D7 parent directive and orch role |

**Entries, peers and parent commits checked**

| Checked | Result |
|---|---|
| `reed-slack-relay` (peer, has a record) | Finding 2; otherwise none (its Scope Out excludes cross-hub mail; the shared `orch-role.md` edit is text overlap only) |
| `drift-rework` (peer) | No conflict |
| `codex-creel-hooks` (F18) | No conflict; D6 reminds Codex pane-mail strands, as today |
| `sandbox-closed-area-build` (F9) | No conflict; the interaction is in Open risks |
| `dormant-orch` (F4) | No conflict |
| `pr-review-wait-silent` (B1) | No conflict |
| Parent commits | `lym loom peers` listed none |

**Decision notes**

The board list showed no `Decision for creel-cross-hub-mail:` note, and the record has no `Added after Discussion` entries.

**Tool calls**

None.
