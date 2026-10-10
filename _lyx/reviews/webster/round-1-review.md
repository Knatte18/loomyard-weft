---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: contracts/stencils/claims_test.go:249
    summary: the three new chair-stencil claims rows are indented with three tabs where the surrounding rows use two, so gofmt rejects the file
---

Range: `git log --first-parent --no-merges -p 7f6e976ad..HEAD` (the live generation's rework round 1, `first_card` 10), which holds one commit, `a1895c842`, touching only the chair stencil and `claims_test.go`.

Checked against the plan's card 10 and the focus directive:

- The advisors section's first bullet now forms batches as Step 3 says and sends every batch to every advisor with no advisor owning an area.
  The second bullet settles each question only after weighing every advisor's answer, and counts a never-started, failed or written-off advisor as giving no answer.
  The remaining bullets are byte-identical to the parent, the header comment is unchanged, and the `loom-template-discussion` and `seat-directive-chair` includes are untouched.
- The two new `must` claims quote text the stencil contains verbatim (`send every batch to every advisor`, `after weighing every advisor's answer`) and sit under `## Working with your advisors`, where both phrases occur.
  The `mustNot` on `an area of the design` matches no text of the stencil now, and the repository greps for `an area of the design` and `assign each an area` hit only the claims row itself.
- The diff adds no test function, only three claims rows, so test economy and `test-budget.yaml` are unaffected.
- The diff touches no `loom.yaml` key, `mode_rules` text or `PATTERN.md`.
- `lyx gate test ./contracts/stencils` passes.

### [MEDIUM] New claims rows are mis-indented and fail gofmt

**Location:** `contracts/stencils/claims_test.go:249-251`
**Issue:** The three added rows start with three tabs while their sibling rows in the same `[]claim` literal start with two.
`gofmt -l contracts/stencils/` lists `claims_test.go`, and the parent revision's file is gofmt-clean, so the commit introduces the violation.
A formatting-clean file is the baseline the surrounding code keeps, and a lint gate over Go sources is liable to reject it.
**Fix:** Re-indent the three rows to two tabs, as `gofmt -d` shows.
