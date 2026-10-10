---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/plan/10-chair-batches-to-every-advisor.md:12
    summary: the settle-only-after-weighing-every-advisor's-answer bullet states no bound for an advisor that is written off or never started, so its Intent leaves open whether the chair may settle without that advisor's answer
---

Answer key: the decision record plus `_lyx/loom/rework/round-1/findings.md`, since `round-1/record.json` carries `class: required` and `first_card: 10`, matching the overview's `first_card: 10`.

Focus 1, fidelity: the card carries the finding's whole fix.
The area assignment goes, every batch goes to every advisor, and in autonomous mode the chair settles after weighing all answers, with interactive mode unchanged.
The finding's conditional claims edit (`if any`) is settled by the card's grep, which I reproduced: outside the plan and rework files, `an area of the design` appears only at `contracts/stencils/loom/loom-template-discussion-chair.md:15`.
No other stencil, doc, `doc.go` or Go file holds the area protocol.
The D6 and constraint items that bear on the advisors section are kept.
That covers the interview and ledger records naming the strand name (the two existing claims stay), the interactive bullet, the notes-file and write-off bullet, Step 6, the Go-held `mode_rules` sentence (`internal/loomengine/discussiontable.go:41`, which already says `self-pick what they cannot settle`), the no-ask-the-operator rule and the test budget.
The auto-pick fallback is not new scope: it is today's `A pick made with no usable answer stays an auto-pick.`, carried from D6's second bullet.
The settle-after-weighing wording is licensed by the finding's `settles each question after weighing all advisor answers`.
The recommended-answer-and-alternatives shape is Step 3's own (`loom-template-discussion.md:68-69`), so nothing beyond the record is added.
The open bound is F1.

Focus 3, claims consistency: the three claim strings are substrings of the Intent's rewritten bullets, spelled identically, with straight apostrophes.
`send every batch to every advisor` is in the first bullet, `after weighing every advisor's answer` is in the second, and `an area of the design` is in neither new bullet; `no advisor owns an area` does not contain it.
`TestStencilClaims` reads the raw chair file, where the include is the unexpanded `{{template "loom-template-discussion"}}` line, and `sectionOf` scopes to `## Working with your advisors`, the file's last section.
So the `mustNot` meets only the chair's own section text, and no kept bullet in it says `an area of the design`.
No collision.

Focus 4, verify coverage: `go test ./...` and `go test -tags integration ./...` run `contracts/stencils`.
The section's order matches `loom-template-plan.md:177`: build, vet under `integration`, `tmux` and `llm`, then the untagged and `integration` test tiers.
No finding.

### [MEDIUM] The settle bullet's "every advisor's answer" has no stated bound for a written-off or failed advisor

**Location:** `_lyx/plan/10-chair-batches-to-every-advisor.md:12` (the second-bullet sentence of `**Intent:**`)

**Issue:** The card changes when the chair may settle in autonomous mode.
Today's bullet says only that the batch goes to the advisors before the chair settles it.
The new bullet says the chair settles each question `only after weighing every advisor's answer to it`.
Read literally, that waits on an answer that can never come: from an advisor the kept write-off bullet tells the chair to write off, from one the included seat directive's failure notice removes, or from one listed under `Advisors that never started`.
The Intent covers only the all-or-nothing case, `a pick made with no usable answer stays an auto-pick`.
It does not say what the chair may do when some advisors answered and others were written off or failed.
It also does not name what bounds the wait.
Per the rubric's attack-surface question, the card should state what the chair can now let through, a question settled without a written-off advisor's answer, and what bounds it.
The bound exists: the write-off bullet the card keeps word for word, and the seat directive's `you decide without it` (`contracts/stencils/seat/seat-directive-chair.md:20-21`).
The Intent names neither, and the stencil bullet as planned contradicts them on a literal reading.

**Fix:** Extend the Intent's second-bullet sentence so the rewritten bullet says that an advisor written off or failed, as the seat directive says, counts as giving no answer.
Then `every advisor's answer` means every advisor still answering, and the Intent names the write-off rule and the seat directive as the bound on the wait.
Keep the `must` string `after weighing every advisor's answer` intact in the reworded bullet so the claim still matches.
