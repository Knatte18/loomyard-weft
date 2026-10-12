## Interview

Autonomous run: no operator answered, so every turn below is a self-pick recorded in the Question ledger.

1. Read the board entry and related entries (`orch-split`, `run-message-routing`, `runs-overview`); `orch-owns-the-loop`, `merge-review`, `parent-review-wait` and `task-run-settings` are no longer on the board and have landed (commits `3a627c00d`, `314816cde`, `414d8b713`, `e6e0baf00`), as has `driver-arbiter` (`694f68f90`).
   The note's "when to start" question is moot: `parent-review-wait` has landed and this run is under way.
2. Read the orch role stencil, the parent-review brief and delivery stencils, `internal/parentreview/doc.go`, `PR-Gate`'s row table and the loom template keys.
   Found: parent review delivers through the writer's SendMessage to `ly:orch`; PR-Gate awaits and batten notifies the orch; `lyx loom approve` has no caller guard; batten verbs refuse outside the prime, so an agent in a pair has no way to list other runs today.
3. Scope: both review points, loom only (Q1, Q2).
4. Verdict application: Go applies a verdict file rather than the agent running the approval verbs (Q3).
5. Escalation trigger: substantive overlap only (Q4); the agent may reject on scope (Q5).
6. PR-Gate placement: a new `PR-Align` row (Q6); approval lands without an orch yes (Q7).
7. Peers read verb: yes, `lyx loom peers` (Q8).
8. Config and failure fallback (Q9, Q10).
9. Lens review by six forks; every fork returned a report.
   Duplicates merged: the `parentreview` import set (leanness, pattern-fit), peer geometry (pattern-fit, premises), the durable reject count (premises, scenarios).
   - premises MEDIUM, PR-Gate escalation travels in `ParentNotice`, not batten's notice: fixed in D5 and Notes.
   - premises MEDIUM, keep-alive pings would not fire before delivery: fixed in D4 (clock from opened-at during alignment).
   - premises LOW, approve must not take the cap-reject supersede path: fixed in D4.
   - premises LOW / pattern-fit LOW, peer pair path owner unstated: fixed in D6 (fabric's pair layout).
   - premises LOW / scenarios MEDIUM, reject-once count needs durable state: fixed in D5.
   - goal-scope MEDIUM, after-approval board amendments lost: fixed in D3, D5 and D8 (superseded entries in the report, one queued orch notice).
   - goal-scope MEDIUM, decisions into a frozen running task's body: fixed in D8.
   - goal-scope LOW, PR-Gate auto-landing cannot be withdrawn alone: fixed with `alignment_pr_approve` in D7 and D5.
   - goal-scope LOW, D2 reads not split by point: fixed in D2.
   - goal-scope LOW, unbounded "unsure" escalation: fixed in D3 (named question required, else invalid).
   - board-fit MEDIUM, `runs-overview` shares `battenshed` and docs: fixed in Open risks.
   - board-fit LOW, runs overview cited as existing: fixed (D5 rationale no longer cites it).
   - board-fit LOW, `orch-split` shares the orch stencil and an open question: fixed in Open risks.
   - board-fit LOW, `convergence-plan` intervention logging: rejected, that note is unbuilt and has no logging surface to fit.
   - board-fit LOW, `lyx-halt` interrupting the agent: rejected, `lyx-halt` is unbuilt and D5's pause handling covers today's pause.
   - pattern-fit MEDIUM, variable output set vs `PATTERN-completion-signal`: fixed in D2 (always both files).
   - pattern-fit MEDIUM, `alignment` model open to the local layer hollows the gate: fixed in D7 (closed).
   - pattern-fit MEDIUM, CLAUDE.md production rule changed unnamed: fixed in Scope and D5.
   - pattern-fit LOW / leanness LOW, `parentreview` import set: fixed in D1 (seam wired in loomcli).
   - pattern-fit LOW, parent directive vs no messages: fixed in D1.
   - pattern-fit LOW, `PATTERN-github-auth` and `PATTERN-hubforge-fixtures`: fixed in Constraints.
   - pattern-fit LOW, idle lock release during `PR-Align`: fixed in D5.
   - scenarios MEDIUM, forged verdict by a live writer or driver: fixed in D2 (per-spawn token; residual stated).
   - scenarios MEDIUM, liveness hold and `noReviewer` order: fixed in D4 and an acceptance criterion.
   - scenarios MEDIUM, restart during parent-review alignment: fixed in D4 and an acceptance criterion.
   - scenarios MEDIUM, parent verdict or notify racing the agent: fixed in D4.
   - scenarios LOW, pause, goto and teardown during `PR-Align`: fixed in D5 and an acceptance criterion.
   - scenarios LOW, coherence on resolved values: fixed in D7 and an acceptance criterion.
   - scenarios LOW, peers with no board, self or non-running slug, branchless peer: fixed in D6.
   - scenarios LOW, does an orch reject count: fixed in D5 (it does not).
   - scenarios LOW, double escalation: fixed in D8.
   - leanness MEDIUM, Constraints restating PATTERN: fixed (task-specific bullets kept).
   - leanness LOW, shared write path as a note: fixed (moved into D5).
   - leanness LOW, docs file list: fixed in Scope.
   - leanness LOW, D5 bound repeated in Open risks: fixed (risk points at D5).
   - leanness LOW, Auto-mode assumptions repeating rationale: fixed (one line each).

## Rejected alternatives

- **Agent runs `lyx loom review approve` and `lyx loom approve` itself.**
  The verbs have no caller guard, so the trust boundary would rest on the stencil alone, and Go could not tie the approval to the head it was spawned for.
- **Spawn the agent from the LLM driver, as the arbiter is.**
  Parent review happens inside Discussion-Write's gate in Go, and `PR-Gate` is a Go row; a driver turn per review adds cost and a second place that can forget it.
- **Fold the spawn into `PR-Gate` itself.**
  `PR-Gate` already owns lock, reroute and nine outcome rows; a long agent wait there would be invisible as a stage and would re-run on every re-call.
- **Wake the orch on any shared file.**
  Bookkeeping files such as the test budget are shared by nearly every run, so this would wake the orch on almost every review and defeat the goal; Merge-In and Merge-Review settle them.
- **Escalate every finding instead of rejecting.**
  Keeps the orch in the loop for plain scope deviations, which today's brief already lets a fork settle on fixed rules.
- **Approved PR waits for a short orch yes before landing.**
  Every run would still cost one orch wake, which is the cost this task exists to remove.
- **Agents read peer worktrees by path.**
  Each agent would form another pair's geometry; the board note asks for a verb instead.
- **Extend darn's PR-Gate too.**
  The board scopes the agent to loom, and darn's PR-Gate is named as the operator's review point.
- **Sonnet as the default model.**
  The operator finds Sonnet weaker than Opus, and the agent replaces an Opus orch judgment.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick, resolved): Which review points? Both parent review and PR-Gate, as the board entry names.
- Q2 (auto-pick, resolved): darn too? No; loom only, since the board says "in loom" and darn's PR-Gate is the operator's review point.
- Q3 (auto-pick, resolved): Who applies the verdict? Go, from a verdict file tied to the spawn and, at PR-Gate, to the head and pull request number.
- Q4 (auto-pick, resolved): What escalates? A substantive overlap with a run in flight, a needed operator decision, or a named question it cannot settle, an escalation without one being invalid; a bookkeeping-file overlap is reported but does not escalate.
- Q5 (auto-pick, resolved): May the agent reject? Yes, on a scope deviation; at PR-Gate once per run, then it escalates.
- Q6 (auto-pick, resolved): Where at PR-Gate? A new `PR-Align` row between `Publish` and `PR-Gate`.
- Q7 (auto-pick, resolved; board open question): Does an agent-approved PR land without an orch yes? Yes, per the operator's 2026-10-12 comment that most traffic should stay inside loom.
- Q8 (auto-pick, resolved; board open question): Do in-flight records need a read verb? Yes, `lyx loom peers`, readable from a pair.
- Q9 (auto-pick, resolved): Config? `alignment`, `alignment_timeout_min`, `skip_alignment`, `alignment_pr_approve` in `loom.yaml`; the timeout and the off switch open to the local layer, the model and the PR-Gate landing closed; timeout must be below `parent_review_wait_min`.
- Q10 (auto-pick, resolved): Failure of the agent? Every failure falls back to today's orch path.
- Q11 (auto-pick, resolved): Default model? `opus[high]`, as the arbiter's.
- Q12 (resolved): The note's "when to start" question: moot, this run has started after `parent-review-wait` landed.
