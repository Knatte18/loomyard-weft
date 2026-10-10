# Card 16 — gate-route-stencils

**Edit:**
- `contracts/stencils/webster/webster-body-verify-fix.md`
- `contracts/stencils/claims_test.go`
- `CLAUDE.md`

**Intent:**
Name the slotted route in the two remaining agent-facing places (D8).
The verify-fix fork's rule that runs the failing packages' tests with `-tags integration` becomes `lyx gate test --tags integration <the findings' packages>`, run as its own Bash call with `run_in_background` and waited for, and a slot-busy exit is re-run, never counted as a failure; `claims_test.go`'s verify-fix claim follows that sentence.
`CLAUDE.md`'s push rule runs `lyx gate test ./...` and `lyx gate test --tags integration ./...`, each as its own background Bash call, in place of the raw `go test` pair.
This branch does not carry main's commit `0cdd14ac9`, so the card writes main's second line itself, directly under the rewritten bullet and verbatim, indented as the bullet's continuation, so Publish's merge-in meets identical text: ``A commit that changes only text no build or test reads, such as `docs/` and this file, is pushed without the test run.`` (decision "CLAUDE.md push rule keeps the text-only line").
No orch stencil carries a test step, so the orch reads the push rule from `CLAUDE.md` alone.

**ImpactSummary:** Two prompt texts and one claims pin; the verify-fix fork and anyone pushing to `main` now route module-wide test runs through the slot.
