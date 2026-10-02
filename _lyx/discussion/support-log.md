# Support log — parent-review

## Interview

The session ran autonomously, so every turn below is a question the agent posed and answered itself after reading the code.

1. **Is the board entry the whole scope?**
   The brief is the operator's 2026-10-02 reshaping of the older `orch-discussion-review` note: the review moves from a Discussion-Review round-1 `awaiting` stop into Discussion-Write's own gate.
   The old note's second mechanism (live questions) is now `discussion-advisor`.
   Scope is the gate, the verbs, delivery, status and the board-reading stencil change.
2. **Can a gate closure wait minutes?**
   `shuttleengine.Gate` is synchronous and the wait loop only evaluates on a Done arrival.
   A blocking closure would stop liveness and deadline checks, so the gate contract gains a pending state that the loop re-polls (D2).
3. **Does `attempts: 1` with `pass_on_cap` give run-once semantics?**
   Read `evaluateGate` and the re-prompt branch: the first failure re-prompts (count 0 < 1), and the next arrival is let through (count 1 ≥ 1).
   Yes (D1).
4. **How does Go notify the parent?**
   The ly-drive skill uses Claude Code's `SendMessage`, which Go cannot call.
   Reed can type into a strand by guid, and `lyx reed list` already enumerates every worktree's strands, so hub-wide resolution by full name plus `SendText` gated by an idle probe works for any reed-hosted parent (D5).
5. **What if the parent is not a reed strand?**
   A hand-started hub is not a strand.
   Delivery fails loudly but does not stop the run; status shows the wait, and the wait bound lets the run through (D5, D9, D10).
6. **What survives a restart?**
   The gate-list commit notes that in-memory counts reset after an attach.
   The closure therefore keys on the on-disk round: an open request, a verdict and a consumed marker (D3, D4).
7. **Where do the files live?**
   Not under `_lyx/discussion/`, which `discussionparser` owns and whose contents the Plan producer is told not to read beyond the decision record.
   Beside the Discussion-Review run directory, committed with the discussion pair (D3).
8. **Where does the reviewer run, and how does it address the child?**
   The hub's fork runs in the prime, so the verbs take a slug and resolve the task worktree through the hub layout (D6).
9. **Where does a writer answer a finding it disagrees with?**
   The support log's `## Review rounds`, which Discussion-Review already reads (D8).
10. **How does the hub see a waiting child?**
    Through `lyx batten status`, since batten's InnerRun watches the child's status file (D9).
11. **Does the writer need `lyx board notes list` as well as `lyx board list`?**
    `lyx board notes --help` says it is an alias over the same store, so `list` alone is enough (D11).
12. **What bounds the wait?**
    `discussion_timeout_min` is 480 minutes; a separate `parent_review_wait_min` of 60 keeps an absent hub from holding a task for hours (D10).

## Rejected alternatives

- **Blocking gate closure that polls internally.**
  It is simpler, but it freezes the wait loop's liveness and deadline checks for the whole review, and `finalize`'s afresh evaluation would block too.
- **A separate recipe row or a separate fixer for the review.**
  The operator ruled this out on 2026-10-02: the live writer fixes with its own context.
- **Discussion-Review round 1 as an `awaiting` stop with two consecutive approves to converge (the older `orch-discussion-review` shape).**
  It is superseded by the operator's 2026-10-02 shape; it would also cost a fresh fixer without the writer's context.
- **The writer sending the notice with `SendMessage` (re-prompting it to notify).**
  It works for any named Claude session, but it bends the fixed gate re-prompt into a messaging channel and spends a writer turn on plumbing.
  Delivery belongs in Go, where it is testable.
- **Routing the notice through the orch watcher as the single typist into the hub pane.**
  It removes the typing race, but works only when the parent is the orch strand, and couples loom to orch's state.
  Reed delivery covers any reed-hosted parent; the race is recorded as a risk.
- **A blocking refusal when the parent cannot be reached.**
  The review is not a correctness guard, so the Refusal Way-Forward Invariant's warn-and-record rule applies.
- **A configurable reviewer address.**
  Only the parent is wired; a config knob for other reviewers is YAGNI.
  The request records a reviewer name so another source can be added later without a format change.
- **Allowing pending from any gate entry.**
  A pending must-pass entry at a deadline would need its own pass-or-fail rule; restricting pending to `PassOnCap` entries keeps every must-pass gate's semantics unchanged.
- **Putting the wait bound on the gate entry in the recipe.**
  It would widen the gate element schema for one entry; the bound is a tunable like the other `*_timeout_min` keys, so it goes in `loom.yaml`.
- **Approve carrying NITs for the writer.**
  It would need a second re-prompt path on approve; a review with findings is a reject, and `pass_on_cap` still lets the next arrival through.
- **Parent review in Plan-Write or Plan-Review round 1.**
  It is outside this board entry's title and shape; the old note left it open.

## Review rounds

_No rounds yet._

## Question ledger

- **Q1 — Gate mechanism for a long wait.** Auto-pick: pending gate state (D2). Resolved.
- **Q2 — Notification channel.** Auto-pick: reed delivery by full name with the idle probe; warn and record on failure (D5). Resolved.
- **Q3 — Verb names.** Auto-pick: `lyx loom review notify|approve|reject`, mirroring PR-Gate's `approve`/`reject` without colliding with them (D6). Resolved.
- **Q4 — File location.** Auto-pick: a per-round directory beside the Discussion-Review run directory, committed with the discussion (D3). Resolved.
- **Q5 — Wait bound.** Auto-pick: `parent_review_wait_min: 60` in `loom.yaml`; expiry lets the run through with a Warn (D10). Resolved.
- **Q6 — Where the writer answers a disputed finding.** Auto-pick: the support log's `## Review rounds` (D8). Resolved.
- **Q7 — Board reading.** Auto-pick: `lyx board list` plus `get`; `notes list` dropped as an alias over the same store (D11). Resolved.
- **Q8 — Rounds per run.** Auto-pick: one round per Discussion-Write start; attach continues it (D3). Resolved.
- **Q9 — Plan-side parent review.** Out of scope for this task. Open for a separate board entry.
- **Q10 — Hand-started hub.** It gets no push notice under this design; the risk is recorded. Open until the hub is hosted by `lyx orch start` as a matter of course.
