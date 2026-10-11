---
verdict: APPROVED
findings:
  - id: F1
    severity: NIT
    class: consistency
    location: internal/loomcli/renumber.go:154
    summary: the invalid-slug way forward in renumberWaysForward says "pass a slug of lowercase letters..." while its refusal-spec row says "pass lowercase letters..."
---

The diff implements the five cards of the plan.
`lyx gate test` over `./internal/planparser`, `./internal/loomcli` and `./internal/websterengine` passes.

What I checked against the focus directive, each with a verdict of no defect:

- **Apply steps and journal.**
  `applyRenumber` computes every new byte in memory, writes the journal through `writeRenumberJournal` (temp file, then `Rename`), and runs the steps in an order that never overwrites an unmoved file: descending for an insert, ascending for a remove.
  Every step failure goes through `reverseRenumber`, which removes the created names before it writes the recorded files back, so a name that is both created and journaled ends as its old bytes.
  A failed reversal keeps the journal and returns `ErrRenumberReversalFailed`.
  `TestRenumber_IOFailureReversed` has one row per distinct step, plus the in-reversal failure for both verbs.
- **`RestoreRenumber` as sole journal reader.**
  `precheck` in `renumber.go` only tests for the journal's existence through `ReadFile` and never decodes it; `readRenumberJournal` is in `restore.go` alone.
  The name check runs before any write, and a failed restore keeps the journal for a re-run.
- **Verbs, bound and lease.**
  `runCardVerb` takes the lease, reads the run record, compares against `websterengine.LastBegunCard`, renumbers, then releases before printing.
  Every refusal is worded in `renumberRefusal`, and each row in the refusal table has a matching case in `TestRenumberVerbs_Refusals`.
- **`LastBegunCard`.**
  It reads every record's `Cards`, terminal or not, with the key fallback for records without ids, which matches the begun set `DispatchScope` derives and not `DoneCards`.
- **PATTERN invariants.**
  `PlanScratchDir` joins `lyxdirs.DotLyxDirName`; no new file spells `_lyx` or `.lyx`.
  `pathrules_test.go` classifies both new paths as transient with `PlanScratchDir` mirroring `PlanDir`, and `PATTERN-sole-parsers` lists the new write paths.
- **Test economy.**
  The new tests are table-driven at the public surfaces (`InsertCard`, `RemoveCard`, `RestoreRenumber`, the verb functions, `LastBegunCard`), and the budget counts rise by exactly the new top-level tests.
- **Stencils and docs.**
  The writer stencil and the review rubric carry the paired reachability rule with the same exemptions; the rework and writer stencils name the verbs; the `doc.go` files, `docs/overview.md` and the plan spec match the verbs' behavior.

### [NIT] Slug way-forward wording differs from the refusal-spec row

**Location:** `internal/loomcli/renumber.go:154`, `contracts/specs/refusal-spec.md` (row `insert-card: invalid slug`)
**Issue:** the code's step reads "pass a slug of lowercase letters and digits joined by single hyphens"; the table row reads "pass lowercase letters and digits joined by single hyphens".
The two other rows with a hand-worded step (`restore-renumber: no journal`) differ by punctuation only.
**Fix:** make the code string and the spec row identical.
