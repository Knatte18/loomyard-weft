# Support log — publish-review-fixback

## Interview

Autonomous session: no operator answered.
Each turn below is a question the design had to settle, the evidence read, and the self-picked answer.

1. **Is this Tenter?**
   `manifest/designs/hardener.md` defines Tenter as a behavior-based reviewer that drives a live substrate inside the Hardener campaign, post-loom.
   A PR-review fix-back is a text-findings → cards → implement loop.
   Picked: a smaller reroute inside loom's recipe; Tenter untouched (D1).
2. **How does the run get from Publish back to Webster?**
   `shedengine` routes only by `on_done`/`on_stuck`, outcomes are closed at three, and `on_stuck` must stay inside one segment.
   Publish's current `Stuck` covers merge conflicts, verify failure and push rejection, which must keep escalating.
   Picked: split Publish; a new gate row owns the review wait and its `Stuck` means only "rejected" (D2, D3).
3. **Who turns findings into Webster work?**
   Webster forks per batch, batches come from plan cards via the batcher, and planparser is the sole parser.
   Picked: a dedicated rework LLM row appends cards to the existing plan under the existing plan-format gate (D5).
4. **Will Webster accept the extended plan?**
   `runlevel.go` refuses a changed plan fingerprint unless `--fresh`, which discards completed batches; the package doc requires every sanctioned rewrite to be re-baselined by its writer.
   Resume already skips completed batches and clears stale outcome, summary and integration report.
   Picked: the rework row calls an exported re-baseline; no rework special case in Webster (D6).
5. **Where do the findings live?**
   `approve` writes an ephemeral scratch record and never commits; the Fabric Git Invariant keeps weft commits in Go at a loom boundary.
   Picked: `reject` writes an ephemeral record with the findings inline; the rework row writes them durably and commits them with the cards (D4, D5).
6. **What happens to Describe and the PR body?**
   Publish today pushes to an open PR but never edits it.
   Picked: Publish edits title and body from the description when changed (D7).
7. **Batten.**
   `internal/battenshed/innerrun.go` resumes an awaiting child only on an approval record.
   Picked: treat a rejection as an operator decision with the same once-per-identity resume (D8).
8. **Plan-Review over rework cards?**
   Picked: skip; Webster-Review and the operator's PR review still judge the diff (D9).
9. **Edge cases.**
   Stale records (head moved), closed PR, malformed record, budget exhaustion, crash between commit and record deletion, zero cards, edits to prior cards.
   Each has a verdict in D2, D3, D4 or D5, or an open risk.

## Rejected alternatives

- **A fourth shedengine outcome (e.g. `Rework`) with an `on_reject` edge.**
  Generic-engine change to a closed outcome set pinned across the recipe spec, parser, validity checker, progress rendering and ly-drive, for one use; the split gets the same routing with existing vocabulary.
- **`reject` (or a `loom start` pre-step) rewriting the status file's current producer to the rework row.**
  Puts a routing edge outside the recipe graph, where the routing tests and progress rendering cannot see it, and makes a verb write driver-owned state.
- **Publish gains `on_stuck: <rework row>` with no split.**
  Every Publish failure (merge conflict, verify, push) would bounce into rework instead of halting for a human.
- **Master plans findings into batches itself.**
  Master is an orchestrator; batches derive from cards, and a Master-written plan edit is exactly what the staleness guard refuses.
- **Rework cards in a separate plan directory with its own Webster state.**
  Splits the card numbering and the `N: <name>` commit trail, and doubles Webster's state handling.
- **Rework cards go through the Plan-Review segment.**
  Adds a review round over operator-authored findings whose diff is reviewed twice more anyway.
- **An append-only check folded into the re-prompting gate.**
  A new gate kind requires a new self-check verb and parity test under the Gate Self-Check Parity Invariant; the stencil states the rule and a post-gate Go check halts on the rare violation.
- **No bounce cap on the new segment.**
  Every other segment is capped; the cap costs nothing here because `approve` still works at an exhausted gate.
- **Fetching findings from GitHub PR review comments, or posting them to the PR.**
  Not requested; the review file is the stated input.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 Tenter or smaller reroute? — resolved, auto-pick: smaller reroute (D1).
- Q2 Routing mechanism back from Publish? — resolved, auto-pick: split Publish into Publish + gate row (D2).
- Q3 Which halts at the gate are `Awaiting` vs `Stuck`? — resolved, auto-pick: only a matching rejection is `Stuck` (D2).
- Q4 Bounce budget for the new segment? — resolved, auto-pick: 5, and `reject` refuses when exhausted (D3, D4).
- Q5 Who converts findings to Webster work? — resolved, auto-pick: rework LLM row appending cards (D5).
- Q6 How does Webster accept the extended plan? — resolved, auto-pick: sanctioned fingerprint re-baseline by the rework row (D6).
- Q7 Where are findings stored? — resolved, auto-pick: ephemeral pending record, durable per-round copy committed by the rework row (D4, D5).
- Q8 Does Publish refresh the PR description? — resolved, auto-pick: yes, when changed (D7).
- Q9 Batten behavior on reject? — resolved, auto-pick: resume once per rejection (D8).
- Q10 Plan-Review over rework cards? — resolved, auto-pick: no (D9).
- Q11 Row and segment names? — resolved, auto-pick: working names `PR-Gate`, `PR-Rework`, `PR-Review`; plan writer may rename.
- Q12 Does `reject` post findings to the PR? — resolved, auto-pick: no.
- Q13 Hand-made fixes outside loom after the split? — resolved, auto-pick: operator pushes them before `approve`; recorded as an auto-mode assumption.
