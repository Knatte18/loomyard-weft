---
round: 1
focus:
  - "internal/boardengine/task.go: does the extracted ValidateEntrySlug keep every slug rule NewTask applied before (length limit, root reservation, error text), so no caller of NewTask changes behavior?"
  - "internal/loomcli/raddlecontext.go and internal/raddleparser/check.go: does the ValidateSlug hook reach every slug a filing makes (part slugs, the root, pair slugs), and does a nil hook or a refused slug fail as slug-invalid without a panic or a double report?"
  - "internal/raddleparser/check_test.go and fixture_test.go: did the rewrite of existing TestCheck rows widen or drop any assertion that exists at the review base, and do the new rows pin the length-limit boundary with literal values rather than computing them from boardengine?"
  - "contracts/specs/raddle-proposal-spec.md and the two doc.go files: does the changed prose match what the code now checks?"
  - "Commit trailers: does the Root-Cause trailer name the defect the diff removes, and does the change description (if present) claim anything the diff does not bear out?"
---

Seed focus drawn from the diff stat and the decision record, which holds only the board entry and one waiver of the red-first gate.
