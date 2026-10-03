# Support log — review-finding-class

## Interview

Autonomous session: no operator answered.
Each turn below is a question the design raised, the evidence read, and the auto-pick.

1. **Is this the right thing to build?**
   The board entry records the rework-generations run (2026-10-02): one fresh round, eight findings fixed, the judge checked fixes against the artifacts and approved with no second fresh review.
   The judge stencil opens by calling the judge a reviewer, and `bouncer.go` maps APPROVED straight to Done.
   The fault is the prompt's role, as the operator said; building the narrowed judge is right.
2. **What does the judge read?**
   Board: facts, not artifacts or code.
   The review file is neither, and the ledger and focus file need finding locations and summaries, so it stays.
   The fixer report goes: reading "all fixed" is the exact failure.
   The rubric stays only for the focus file's Do-not-flag check and the class meanings.
3. **Where do the facts live?**
   A Go-written `round-<N>-facts.md` in the run dir, so per-class counts are durable and visible after the run (the board's "round record"), and the judge keeps reading files rather than a prompt-embedded table.
4. **How does "end the loop at zero gating-class findings" fit "the judge decides all three outcomes"?**
   The board says only two things stay mechanical, so the class rule becomes the judge's written default (D6), with a departure allowed only by naming the finding.
5. **Which classes gate, and per segment?**
   The draft design doc says only `design` gates.
   One gating class everywhere, each rubric defining `design` in its catchment, is simpler than per-rubric gating config nobody asked for.
6. **Does a BLOCKING finding of a non-gating class stop convergence?**
   Yes: a reviewer's BLOCKING is the strongest signal, and class labels are as noisy as severities.
7. **How does circling escalate, and how does the operator get out?**
   Shed's Awaiting is the planned human hand-off.
   `lyx shed goto` from an awaiting row moves only strictly before it, so at a Bouncer the only existing exit would be a full rewrite.
   Two recorded-decision verbs, modelled on PR-Gate's approve/reject records, give continue and accept.
8. **What stops an accept from settling a later generation?**
   The accept must be recorded as settled, so re-entry clears and re-seeds like after `CONVERGED`; the mechanism is the plan's, the requirement and its test are in the decision record.
9. **Which model does the seed use?**
   The seed is a Bouncer pass; one key per row is simpler, and its focus never caps the opus reviewer.
10. **What about in-flight runs when the binary updates?**
    Three runs are in Discussion-Write now.
    Read-only verdict aliases and fail-soft facts rows cover a segment straddling the update.
11. **Board context.**
    `pattern-init` depends on this task and will retire `manifest/` and convert model keys to short form; this task writes `judge` in short form and deletes only its own draft doc and roadmap entry, so the two do not collide.
    `convergence-plan` owns the reviewer model trial; untouched here.

## Rejected alternatives

- **A pure mechanical convergence rule in Go** (stop at zero MEDIUM-or-worse): rejected by the operator on 2026-10-02; severities are noisy, a nitpicky MEDIUM forces rounds to `max_bounces`, a reworded finding looks new.
- **Keep APPROVED/BLOCKING and add CIRCLING**: keeps the reviewer framing that caused the defect.
- **No legacy verdict aliases**: a run dir from the old binary would parse as unjudged, and on rework re-entry an old approved round would be re-judged instead of cleared, letting a rewritten artifact pass on a stale review.
- **Facts embedded in the judge prompt only**: not durable, so recurring classes are invisible after the run.
- **Per-rubric gating classes**: configuration nobody asked for; each rubric already narrows `design`.
- **Only `design` at MEDIUM-or-worse blocks, BLOCKING of other classes ignored**: drops the reviewer's strongest signal.
- **Circling as a blocked halt (Stuck with budget)**: a Bouncer's Stuck always routes to the round producer; making it block needs a new Shed feature, and Awaiting already exists for human hand-offs.
- **Circling resolved by resume alone (resume means continue)**: no accept path, and a stray resume would silently spend a round.
- **Accept by rewriting the judge's verdict file to CONVERGED from the verb**: the next Call would read it as settled earlier and clear the segment instead of settling it.
- **Seed on the `review` key, judge on `judge`**: two model triples per Bouncer row for little gain.
- **Go-side ledger carry-forward enforcement**: a known soft spot, but not asked for; the facts read the ledgers without judging them.
- **Including treadleengine's judge**: not a loom review segment; the board names three segments.

## Review rounds

_No rounds yet._

## Question ledger

| # | Question | Status | Answer |
|---|---|---|---|
| Q1 | Judge verdict vocabulary | resolved, auto-pick | `CONVERGED`/`CONTINUE`/`CIRCLING`, legacy words read as aliases (D1) |
| Q2 | Judge reading list | resolved, auto-pick | facts file, latest review, previous ledger; rubric for focus only (D2) |
| Q3 | Facts format and location | resolved, auto-pick | Go-written `round-<N>-facts.md`, fail-soft rows (D3) |
| Q4 | Class required or optional | resolved, auto-pick | required, fail-loud like severity (D4) |
| Q5 | Gating classes | resolved, auto-pick | `design` only, all segments (D5) |
| Q6 | Mechanical vs judged stop on gating classes | resolved, auto-pick | judge's written default rule with named departures (D6) |
| Q7 | Circling escalation and exit | resolved, auto-pick | Awaiting plus `lyx loom circling accept` and `lyx loom circling continue` (D7) |
| Q8 | Does `continue` count against `max_bounces` | resolved, auto-pick | yes; budget block's goto gives a fresh budget (D7) |
| Q9 | Seed model | resolved, auto-pick | `judge` key with the judge (D8) |
| Q10 | Fate of the draft design doc and roadmap entry | resolved, auto-pick | both removed, content into burlerengine's package doc (D9) |
| Q11 | Verb name `lyx loom circling` | resolved, auto-pick | free in `lyx loom`'s namespace; avoids `review` (parent-review) and `approve`/`reject` (PR-Gate) |
