# Support log — loom-bouncer-rounds

## Interview

The session ran autonomously; no operator answered.
The board entry already carried the operator's decisions of 2026-10-05 (checkpoint judging, both numbers as configuration, escalation to the parent, the decision-record verb, no GitHub issue for a recurring finding).
Each turn below is a question the interview would have put, with the self-picked answer.

1. Scope check — is this the right thing to build?
   The `board-model` halt was settled by the hub with five design calls and no operator input, so routing the decision to the parent fits the evidence.
   Kept as scoped; `treadleengine`'s own `CIRCLING` judge left out because it has no consumer.
2. Checkpoint: one round or every round from it on?
   Picked every round from the checkpoint on (D1), so a loop that stalls after round 3 can still escalate before the budget.
3. What is "no progress"?
   Picked: a gating finding on a ledger key open in the judged round and an earlier round, unchanged or reopened; a Go guard on the judge's own ledger enforces it (D2).
4. Where do the checkpoint and budget live?
   Picked `loom.yaml` for both, one key each, applied uniformly to the Bouncer review segments; `PR-Gate` keeps its recipe budget (D3).
5. How does a spent budget escalate without changing Shed's generic block?
   The Bouncer escalates before returning the `Stuck` that would block, using its existing `Bounces` seam; a `continue` buys one exempt round (D4).
6. Who delivers the message to the parent?
   The Bouncer's judge has exited and Go has no SendMessage, so the `ly-drive` driver relays a Go-rendered notice carried on the `awaiting` envelope (D5, D6).
7. What does the parent's fork do, and how does the run resume?
   It records design calls with the new verb, records accept or continue with the existing `circling` verbs, then runs `lyx loom start` in the task worktree (D7, D9).
8. Shape of the post-Discussion verb?
   `lyx loom decision add` with flags, append-only, validated, committed, implemented in `discussionparser` (D8).
9. Smoke failures: stale tests or regression?
   Reading the code: tests compare strand names by equality while production uses `agentname.Matches`; presumed stale, confirmed by running at Plan time (D12).
10. Testing per module: untagged tests for the guard, config, verb refusals and anomaly removal; the smoke tier for D12 (acceptance criteria).

## Rejected alternatives

- CIRCLING allowed at exactly one checkpoint round: a loop that stalls after it could only end at the budget.
- Keeping the checkpoint and budget in the recipe: the recipe is embedded and needs a rebuild, against "configuration".
- Per-segment checkpoint and budget keys: no request for per-segment tuning.
- Keeping `max_bounces` on review rows with a `loom.yaml` override: two sources of truth for one number.
- Teaching `shedengine` to escalate on budget exhaustion: puts review-loop policy into the generic engine.
- A parent `continue` on a spent budget granting a fresh full budget: one answer would open an unbounded loop; `lyx loom goto` already does that deliberately.
- Reusing `internal/parentreview`'s request/delivery/verdict store: it is built around a live writer session that delivers; the Bouncer has none, and the existing circling decision file already records the verdict.
- Go sending the notice into the parent's tmux pane through reed: risks mid-turn sends (the `parent-review-hardening` note) and would put provider wording outside stencils.
- Having the judge agent send the message: the judge is a clean-room single-call agent with no parent knowledge.
- Making the circling verbs resume the run: they deliberately resume nothing, matching `lyx loom approve`.
- Extending batten to resume on a circling decision: unnecessary once the fork runs `lyx loom start`.
- A free-form markdown file argument for `decision add`: flags keep the appended entry's shape fixed and validatable.
- Keeping the recurring-finding anomaly but routing it to the parent: the escalation already carries the recurring keys.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 Checkpoint semantics — resolved, auto-pick: every round from the checkpoint on (D1).
- Q2 Circling evidence — resolved, auto-pick: open key spanning the judged round and an earlier one, Go-guarded (D2).
- Q3 Config location — resolved, auto-pick: `loom.yaml`, `review_circling_checkpoint` default 3 and `review_max_bounces` default 5 (D3).
- Q4 Budget-escalation continue — resolved, auto-pick: one exempt round per decision (D4).
- Q5 Delivery channel — resolved, auto-pick: `ly-drive` relays `parent_notice` to `LYX_PARENT` (D6).
- Q6 Resume — resolved, auto-pick: the fork runs `lyx loom start` in the task worktree (D9).
- Q7 Verb shape and refusals — resolved, auto-pick (D8).
- Q8 Smoke cause — resolved provisionally, auto-pick: stale tests, confirmed by running at Plan time (D12).
- Q9 Config reach to the parent (`config-reaches-parent`) — open, recorded as a risk.
- Q10 Ordering against `board-labels-and-skills`, which moves `ly-drive` into a stencil — open, recorded as a risk.
