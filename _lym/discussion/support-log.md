# Support log — review rounds converge

## Interview

Autonomous run: no operator answered; every pick below is an auto-pick recorded in the question ledger.

### Turn 1 — exploration

- Board: the entry's three ideas (rebuttal, incremental re-review, stalemate) are the goal.
  No `Decision for review-rounds-converge:` note exists.
  Touching entries: `unrequested-scope-lens` (a review lens, no overlap with round mechanics),
  `edit-loop-detector` (shuttle edit counting, no overlap), `orchestrator-survey-ideas` (source note).
- Burler today: the fixer-report is prose with `Disputed`, deferred and `Rulings` sections, parsed by no Go code.
  The next reviewer gets prior review and fixer-report paths with a clean-room rule: form own findings first, read prior files after.
  So a rebuttal exists in prose but reaches the next reviewer only after its review is formed, and never beside its finding.
- Each round reviews the whole target again.
- The Bouncer's causes are `circling`, `budget`, `hard-cap` and `decision`; `circling` rests on the LLM judge's verdict, guarded by Go evidence.
  No code records the target's tree state per round.
- Fixer scopes: `source` (warp, commit per fix) and `overlay` (`_lym` documents, the loop commits).
  The stalemate check therefore needs a target id for both, not only the warp tree the entry names.

### Turn 2 — design, self-decided

- Rebuttal: structured frontmatter on the fixer-report (D1), rendered by Go beside each finding (D2), answered in the review's `prior` list (D3).
- Incremental re-review: `prior` statuses plus `carries` (D3), a changes file and a Go check bounding new findings (D4).
- Stalemate: a per-round snapshot record (D5) and a fifth escalation cause settled by the existing circling verbs (D6).
- The entry says "warp tree"; the design generalizes to the review target so Discussion-Review and Plan-Review, whose fixers edit `_lym` documents, get it too.

### Turn 3 — lens review

Every lens fork returned a report.

- interactions, bound-breaker (BLOCKING, merged): the stalemate check ran before the judge, though convergence and hard cap are decided after it. Fixed: D6 runs in the unconverged path after the judge, before circling, soft-cap and hard-cap rules.
- walkthrough (MEDIUM): skipping the judge left a stalemate round without a ledger. Fixed by the same D6 change: the judge runs every round.
- interactions, walkthrough, bound-breaker (MEDIUM, merged): an unruled `decision`-class finding could drop out of later reviews. Fixed: D2 lists it and D3 carries it with `carries`.
- interactions (MEDIUM): the rebuttal channel depended on the diff cache. Fixed: D2 and D3 need only a complete round N-1; only D4's file bound needs the changes file, and without it the BLOCKING rule holds alone.
- bound-breaker (MEDIUM): a `ruling` entry was required though no Go record shows which findings a ruling reached. Fixed: D1 allows it, never requires it.
- bound-breaker (MEDIUM): a new generation's first round was not full. Fixed: D4 names it.
- walkthrough, bound-breaker (MEDIUM, merged): location path form was unstated. Fixed: D4 normalizes to worktree-relative paths up to the first `:`, `#` or space.
- existing-paths (MEDIUM): the review gate passes on its cap, so D4's bound was not absolute. Fixed: D4's bound states the cap and that the fixer works what passes.
- existing-paths (MEDIUM): Go-written fixer-reports lacked frontmatter, and fixer-less rounds counted as unchanged. Fixed: D1 gives them frontmatter; D6 counts a fixer-less round as changed.
- existing-paths (MEDIUM): `lym burler validate-review` had no round context. Fixed: D3 finds round N-1 from the run directory, else shape checks alone.
- existing-paths (LOW): a review re-run re-took the snapshot. Fixed: D5 writes the record once per round.
- existing-paths (LOW): `retireContinue` moves the decision file the count restart read. Fixed: D6 restarts after the latest `stalemate` escalation round, not a decision file.
- interactions (LOW): a `continue` on another cause restarted the count. Fixed: D6 says only a stalemate escalation restarts it.
- interactions (LOW): round indexing differed between decisions. Fixed: one convention stated above D1.
- walkthrough (LOW): verify output as dispute evidence. Fixed: D1 states the narrowing as deliberate.
- walkthrough, bound-breaker (LOW, merged): untracked files in the source snapshot. Fixed: D5 bounds the id to what `git add -A` stages; D4's bound names the widening.
- walkthrough (LOW): no named source for "reviews an amendment". Rejected: the review producer already renders the round's amendment note into its rubric, a fact for Plan to wire.
- board-scope (LOW): "halts" read as an escalation was unstated. Fixed: D6 says the escalation is how the segment stops.
- board-scope (LOW): "warp tree" widened to the review target unstated. Fixed: the Goal says so.
- leanness (MEDIUM): the sole-parsers background edit sat in Notes. Fixed: moved into Constraints; Notes removed.
- leanness (MEDIUM): the first open risk restated D4's bound. Fixed: cut.
- leanness (LOW): Constraint lines restated D5 and D6. Fixed: trimmed; the key's category now sits only in Constraints.
- leanness (LOW): Auto-mode bullets restated decisions. Fixed: shortened.
- leanness (LOW): D2 narrated today's text. Fixed: trimmed.
- leanness (LOW): vague Notes bullet on the judge. Fixed: cut.
- leanness (LOW): the review-start premise sat in Notes. Fixed: moved into D5's rationale.

## Rejected alternatives

- **Rebuttals left in prose, read by the reviewer from the fixer-report path.**
  That is today's channel; the reviewer reads it only after forming its findings, and Go cannot place a rebuttal beside its finding or check an answer.
- **A separate rebuttal file per round.**
  The fixer-report already holds the dispute; a second file splits one outcome across two writes.
- **Keeping still-open findings only in `prior`, with the fixer reading both lists.**
  It changes the fixer's input and every reader of `findings`; `carries` keeps one list.
- **Hunk-level Go check on new findings.**
  A finding's `location` is free text with an optional line; a line-level match against diff hunks would refuse legitimate findings on a line next to a hunk.
  The file-level check holds a firm bound, and the stencil states the hunk rule.
- **HEAD commit as the source-scope snapshot.**
  An uncommitted fix leaves HEAD unchanged and would read as a stalemate.
- **Stalemate as a halt rather than an escalation.**
  A halt needs a resume path of its own; the escalation and its `accept`/`continue` verbs already give the parent both outcomes.
- **Replacing `circling` with the stalemate check.**
  `circling` also catches rounds that change the tree yet re-raise the same finding; the entry asks for the Go check, not the removal.
- **Stalemate check before the judge call, to save a judge session.**
  Convergence and the hard cap are decided after the judge, so an early check escalated rounds the judge would converge
  and left the round without a ledger.
- **Per-perch stalemate keys.**
  No need for different counts per segment has been seen; one loom key.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 — Where do rebuttals live? Auto-pick: frontmatter on the fixer-report (D1). Resolved.
- Q2 — Does an earlier open finding stay in `findings`? Auto-pick: yes, carried with `carries` (D3). Resolved.
- Q3 — How far does Go enforce "only on code the fixes changed"? Auto-pick: file level in the review check, hunk level in the stencil (D4). Resolved.
- Q4 — Are parent-origin findings bound by D4? Auto-pick: no, the parent's ask sets their scope (D4). Resolved.
- Q5 — Does the stalemate cover overlay segments, whose fixer edits `_lym` documents and not warp? Auto-pick: yes, through a content hash (D5). Resolved.
- Q6 — Working tree or HEAD for the source snapshot? Auto-pick: working tree (D5). Resolved.
- Q7 — Stalemate count default and key shape? Auto-pick: one loom key `review_stalemate_rounds`, default 2, 0 off (D6). Resolved.
- Q8 — New verb for settling a stalemate? Auto-pick: no, the `lym loom circling` verbs (D6). Resolved.
- Q10 — Where does the stalemate check sit against the judge? Auto-pick: after a non-converging verdict, before circling, soft-cap and hard-cap rules (D6). Resolved.
- Q11 — Are unruled `decision`-class findings carried? Auto-pick: yes, listed in D2 and carried in D3. Resolved.
- Q9 — Do cluster and assisted rounds keep their fan in an incremental round? Auto-pick: yes, with the same rule and the review check as backstop (D4). Resolved.
