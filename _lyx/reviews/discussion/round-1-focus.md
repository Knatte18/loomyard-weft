---
round: 1
focus:
  - "D10 and D11, `intake import` and `intake close`: these are the only decisions that mutate a repo outside the worktree. Settle whether each states what it can now do to an issue (comment, close, close reason) and what bounds it, including the `into` fold target (any kind or status?) and the retry path for a recorded open issue (what supplies the slug, what stops it closing an issue the operator did not intend)."
  - "D10 sequencing, board write under lock then GitHub close after: settle whether a failure at each point (issue fetch, board write, comment, close) leaves a state the Decisions and Acceptance criteria both cover, and whether the 'recorded and open' retry rule can misfire on an issue a user recorded by hand in `issues` via D9."
  - "D8, `--body-file`: settle whether the Decision bounds what the flag can read (any path, stdin) and what a body it supplies can skip, given that every other field still passes validation."
  - "D3 against D12 and D13: settle whether migration output (types, `undecided`, stripped prefix labels, the one-type-label rule for notes, one-or-more for tasks) can produce an entry that D3 refuses on its next write for a reason other than a missing config label, and whether the 'refused on next write' path leaves the operator a way forward."
  - "D2 against D12: settle whether migrating `depends_on` (note dependencies into the body, task dependencies on what were tier 2 or 3 entries) can leave a task depending on a note, or a done entry in a state D2's write-time rules would refuse."
  - "D1, D4, D5, D6, D7 and the Acceptance criteria: settle whether any acceptance criterion contradicts a Decision (for example the payload-with-`tier` refusal versus D1/D4, the `Other` README section versus D7, `--label` refusal versus D3's no-validation-on-read), and whether any Decision or Scope item is design deliberation that belongs in `support-log.md` rather than `decision-record.md`."
---

Seed rationale: the record is already lean and carries bounds on D3, D8, D10 and D13.
The residual risk sits in the external-effect path of `intake`, in cross-decision interactions (migration against the write-time rules), and in Decision/Acceptance consistency.
No entry asks for rejected alternatives, a plan-writer subsection or call-site enumeration.
