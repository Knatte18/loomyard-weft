---
round: 2
exclude_lenses: []
focus:
  - "Member checks under the parent lock (D1, D3, D6, D10): for each thing today's Finalize checks or decides for the landing run under the lock (run-scope reconcile and hand-over, owner read, beam land check, pull-request decision), settle whether a member the leader lands gets it, with what data, and what happens to a member that fails. Hold this against PATTERN-owner-machine and the declared PATTERN-landing text."
  - "Window between taking the batch and moving the parent (D3, D6, D7): settle whether a pause, a dead member, a moved task tip or a moved parent head during the gates can still land or lose work, and whether every rebuild of the chain is bounded."
  - "Queue state under .lym and recovery (D2, D6 backstop, D8): with the queue directory lost, an entry pruned, a leader dead before or after the move, or a lock taker that is no run such as lym fabric pull or lym fabric anchor, settle whether each moved commit still gets its ledger landing and whether landed work can be committed to the parent a second time. Hold this against PATTERN-lym-files."
  - "Entry and outcome lifecycle (D2, D3, D8): across Done, a halt, Stuck, a rejoin at Finalize and a lym loom goto back with new commits, settle which entry and outcome survive, and whether any unjudged or culprit head can be batched or any new commit left unlanded."
  - "Records push hold (D2, D6): settle what holds a member's records commits, from when, and whether a push between join and the parent's move can leave the ledger holding a commit origin already has."
  - "Detached leader worktree (D5): settle what every reader of that worktree's HEAD does while it is detached (pair pusher and its refusal read, parent-holds-work check, owner checks, gate scope), how a run resumed or sent to any row gets its branch back, how a batch detach is told from an operator's, and whether ignored build output left by a checkout dirties the batch gate or the restore."
  - "Own-path runs (D7, D8): for a culprit that is the leader or a member, and for a conflicting run, settle whether any fixer or conflict session runs with the parent lock held, whether another leader can take its entry meanwhile, and whether one tree is gated twice."
  - "Acceptance criteria against the Decisions: settle whether each behaviour a Decision adds, including any added since the record was first written, has a named test intent and tier, and whether any criterion or bound states a cause or count the mechanism cannot produce."
---

Every lens that ran had a finding at MEDIUM or worse, so none is excluded.
