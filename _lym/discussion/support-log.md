# Support log — pr-review-row

## Interview

### Turn 1 — exploration, autonomous

The run is autonomous; no operator answers in the pane.
Read the board entry (T2) and the board list, then the code `relevant.md` lists; no other entry plans pull-request work.

### Turn 2 — operator decision via ly:orch (creel #18)

`ly:orch` relayed the operator's decision: the interactive agent may change the branch, since it writes to its own task branch.
What stays open is how its commits are gated and how the pull request is brought level after them.
Answered with `lym creel post --reply 18`; settled as D8 (gate and push by the row) and D11 (Finalize brings the branch level under its lock and pushes before merging).

### Turn 3 — design settled by auto-pick

Every other open point of the entry was settled by auto-pick; see the question ledger.

### Turn 4 — operator decision via ly:orch (creel #20)

`ly:orch` relayed the operator's decision on how a waiting run is seen: it registers a wait marker that battenshed's quiet check already honours, so no quiet notice goes off; no new notice and no change to dobby.
Answered with `lym creel post --reply 20`; D10 now keeps such a marker live and sends no notice, and the parent creel lines at open and end were dropped.

### Turn 5 — lens review

All six forks returned a report.

- walkthrough #1 (BLOCKING), interactions F2: an `entry_gate` cannot read the parent's anchor, a board field or the origin — fixed: the row decides in its call and returns Done on a no (D3, D13), with a `PATTERN-shed-recipe-registry` change.
- bound-breaker #1 (BLOCKING): the board lock freezes the field during the run, so D4's upsert way forward is unreachable — fixed: D2 states the field is fixed while the run holds the entry, and D4's way forward is a GitHub origin or cancel and rerun.
- bound-breaker #2 (BLOCKING), existing-paths D8 (MEDIUM): an agent edit or commit during the gate, and the detached records push, can put an ungated head on the pull request — fixed: the verdict counts only on an unchanged head and clean tree, the push only when the head equals the gated head, and D8's bound now names the detached push, with Done and Finalize reading only the gated head.
- interactions F3, walkthrough #2, bound-breaker #3: Done could leave a dirty tree or let a post-approval commit land — fixed: D9 stops the session, then re-reads head and tree before Done, and resumes the session otherwise.
- interactions F1/F4, walkthrough #3: a stale record outlives a no — fixed: a no closes the open pull request and removes the record (D3), and Finalize follows the resolver, not the record (D11).
- interactions F6, existing-paths LOW: the gate's verify mark clears the `PR #<n>` mark — fixed: D10 sets it again after each gate.
- interactions F5: Finalize's push leaves the record stale — fixed: D11 records the pushed head.
- walkthrough #4: a `COMMENTED` review read as clearing a change request — fixed: D9 defines a reviewer's standing review.
- walkthrough #5: a merge on GitHub by someone else lacks lym's trailer — fixed: stated as a bound in D11.
- walkthrough #6: the beam sentence named an ambiguous `pull_request` — fixed in D2.
- existing-paths D12, bound-breaker #6: only `lym batten cancel` closed the pull request — fixed: D12 moves the close into `internal/pairteardown`, and the `PATTERN-github-auth` text names pair teardown.
- existing-paths LOW: the push must be the blocking one — fixed in D6.
- bound-breaker #4: stopping the session on a cancelled context defeats the reinvoke — fixed: a cancelled context leaves it running (D7), and the interrupt-policy comment changes.
- bound-breaker #5: pause and hibernate during the wait — fixed: each wake reads pause coverage, stops the session and ends the call transient (D9).
- bound-breaker #7: GitHub refuses a merge while it computes mergeability — fixed: a retry of at most a minute in D11.
- bound-breaker #8: `ParseOwnerRepo` misses `ssh://` and user-info URLs — fixed: D4 extends it and halts on a `github.com` origin it still refuses.
- bound-breaker #9: the title and body go stale on re-entry — fixed: D6 sets them on every entry.
- bound-breaker #10: an unreadable parent `beam.yaml` — fixed: D3 halts with its way forward.
- board-scope #1 (decision): the operator cannot approve a pull request opened with their own account — fixed: moved into `## Open risks` as a question for the parent.
- board-scope #2: `off` under `required` is only warned — rejected: within the entry's wording, and halting a run at its landing over metadata costs more than it guards.
- board-scope #3: the diff is not named for the agent — fixed: D7 names the parent branch to diff against.
- leanness #1: rejected-alternative clauses in rationales — fixed where they argued against an option (D7, D10).
- leanness #2: Constraints repeat decisions — fixed in part: restated facts dropped; one-line PATTERN pointers kept, as they cite the entries the plan must obey.
- leanness #3, #4: Auto-mode and Open-risk duplicates — fixed: the default-`on` line stays only in Open risks.
- leanness #5: the shuttle note belongs in D7 — fixed; the docs note dropped, since `CLAUDE.md` already requires it.
- leanness #6: D11 narrated the already-landed mechanism — fixed: D11 cites the existing path only.
- leanness #7: the gh fact sat in D5's rationale — fixed: moved to Constraints.
- leanness #8: Turn 1 listed every file — fixed: it points at `relevant.md`.

## Rejected alternatives

- A new per-worktree config module for the mode: `beam.yaml` already is the per-worktree module of "this beam's tasks", and the prime is the root beam.
- Carrying the override in `run_settings`: it seeds the local layer, which a `CategoryCrossPair` key may not enter, and the entry asks for a board field.
- An `entry_gate` for the mode, or a new entry-gate operand kind: the gate reads `module.key` from the run's own stack, while the mode reads the parent's anchor, a board field and the origin; a new operand kind for one row is wider than the row's own call.
- Halting a run whose field says `off` under a `required` parent: the mismatch is metadata, and `required` already decides the outcome.
- Returning `Awaiting` while the pull request is open: it halts the run and needs a resume after every approval.
- Returning `Yield` from the row: `Yield` is batten's InnerRun verdict, and the loom driver would end on it with nothing to re-step the row.
- Bringing the pull request level with its parent during the wait: the entry gives that to Finalize under the lock, and a merge outside the lock races sibling landings.
- Letting the agent push its own commits: it would put ungated heads on the pull request and give the agent the remote.
- Keeping agent commits off origin entirely: the detached records push carries the task branch, and suppressing it for one row would change fabric's push contract for every run.
- A lym verb for the operator's own approval: an agent with the orch's reach could run it; left to the parent as an open risk.
- Reading `GH_TOKEN` for a named account: a session's environment token belongs to the session's account.
- Restarting the agent on every wake: an operator who closed the window would see it reopen.
- Stopping the agent on a cancelled context: the reinvoke's re-call would find no session and the conversation would be lost.
- Closing the pull request only in `lym batten cancel`: other teardown routes would leave it open.
- Creel lines to the parent at open and end: the operator ruled out any new notice (creel #20).

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (resolved, auto-pick): where does the per-branch mode live? `beam.yaml`'s `pull_request`, read from the receiving worktree (D1).
- Q2 (resolved, auto-pick): what is the override field? Board `pull_request`, empty, `on` or `off`; `required` wins with a Warn (D2).
- Q3 (resolved, operator via ly:orch, creel #18): may the agent change the branch? Yes, on its own task branch.
- Q4 (resolved, auto-pick): how are the agent's commits gated? The row runs the `pr-review` gate on a clean moved head and pushes on a pass (D8).
- Q5 (resolved, operator via ly:orch, creel #20): how does the waiting run show? A wait marker the quiet check honours, no new notice, no dobby change (D10).
- Q6 (resolved, auto-pick, fact checked): which credentials? landing.yaml `pr_account` resolved by `gh auth token --user`, available from gh 2.40.0 and present in the installed gh 2.103.3 (D5).
- Q7 (resolved, auto-pick): what happens on cancel or rejection? Every teardown closes an open pull request with a comment; a closed pull request halts with reopen-and-resume or cancel as the way forward (D9, D12).
- Q8 (resolved, auto-pick): what is the agent given? Paths and identifiers only, read by the agent itself (D7).
- Q9 (resolved, out of scope by the entry): branch protection as the source of `required`.
- Q10 (resolved, auto-pick): what does a hub without a GitHub origin do? A no under a parent-derived `on`, a halt under `required` or a field `on` (D4).
- Q11 (resolved, auto-pick): does the beam's landing get the row? Yes, in the pawl recipe (D13).
- Q12 (resolved, fact): can the operator approve a pull request their own account opened? No: GitHub refuses an author's approval; recorded as a constraint.
- Q13 (open, for the parent): how does the operator approve a pull request opened with their own account? Recorded under `## Open risks`.
- Q14 (resolved, auto-pick): how does the mode skip the row? The row decides in its call and returns Done on a no (D3).
