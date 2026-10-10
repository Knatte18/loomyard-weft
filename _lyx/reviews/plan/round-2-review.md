---
verdict: APPROVED
---

No findings.

The live generation is `_lyx/loom/rework/round-1/`: its `record.json` carries `class: required` and `first_card: 10`, matching the overview's frontmatter, so its `findings.md` joins the decision record as the answer key.
Card 10 covers that file's one finding: it replaces the area assignment with every batch to every advisor, settles after weighing all answers in autonomous mode only, leaves interactive mode word for word, and checks for claims that pin the old wording.
No card's scope goes past what the finding licenses: the overview's `no-area-mode` decision keeps the area mode out, as the finding asks.

## Focus verdict

The directive asked two questions about card 10's reworded autonomous bullet.

Bound on the wait: not a defect.
The Intent says an advisor that never started, sent a failure notice or was written off counts as giving no answer, and that a pick made with no usable answer stays an auto-pick.
It names the bound on the chair's wait as the write-off bullet, which the card keeps word for word, and the rule in `contracts/stencils/seat/seat-directive-chair.md` that the chair decides without an advisor that does not answer in reasonable time.
Those cover all three cases the directive names:
an advisor listed under `Advisors that never started` never answers, so there is nothing to wait for;
a failure notice ends the wait at once;
and a slow advisor is written off by the kept bullet and the seat directive.
The Intent also states what the change lets through: a question settled without the answer of an advisor that failed or was written off.

Claim string: present verbatim.
The reworded bullet reads "settle each question only after weighing every advisor's answer to it", which contains `after weighing every advisor's answer` exactly.
The first bullet's "send every batch to every advisor" also matches its `must` claim.
The new wording "no advisor owns an area" does not contain the `mustNot` phrase `an area of the design`.

## Other checks

- The card's grep claim holds: outside `.git` and `.lyx`, `an area of the design` and `assign each an area` match only `contracts/stencils/loom/loom-template-discussion-chair.md:15`, and the chair row of `wordingClaims` in `contracts/stencils/claims_test.go` has no claim on the old text.
- The section-scoped claims are possible: `claims_test.go` scopes a `section` claim from the heading line to the next `## ` heading, and the existing chair-row claims already use `## Working with your advisors`.
- The claims go into the existing chair row and no top-level test is added, so the card does not need to touch `cmd/lyx/testdata/test-budget.yaml`.
- The Go-held autonomous sentence in `internal/loomengine/discussiontable.go` ("Send your design questions to the advisors named in this prompt, and self-pick what they cannot settle.") does not mention areas and agrees with the new bullet, so `no-area-mode` is correct to leave it alone.
- Verify coverage: `go test ./...` and `go test -tags integration ./...` run `contracts/stencils`, the only package the card targets, and the sections are in the template's order.
