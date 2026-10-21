---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    origin: lens:code-pattern-fit
    location: PATTERN.md:11
    summary: Commit 8902764f silently drops clauses from ten PATTERN entries the declared changes do not list
    fix: In PATTERN.md restore the entries PATTERN-durable-vs-ephemeral-state, PATTERN-hub-wide-config, PATTERN-push-both-sides, PATTERN-driver-choice-single-site, PATTERN-stencil-ownership, PATTERN-friction-capture, PATTERN-edit-directive, PATTERN-testkit, PATTERN-hubforge-fixtures and PATTERN-markdown-link-integrity to their text at the merge base 860c5d03, and keep only the added PATTERN-sparse-hub-base line; if the parent branch changed one of them since, use the parent's current text.
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/fabriccli/clone.go:79
    summary: The cleanup that removes the marker when the clone's board commit fails has no test that forces a commit failure
    fix: Add a row to the clone test in internal/fabriccli/hubmutation_integration_test.go that makes the board commit fail after the marker is written and asserts the marker file is gone; if no failure seam exists, state in the test file why the path is left uncovered.
---

### [BLOCKING] Undeclared edits to ten PATTERN entries

**Location:** `PATTERN.md` (commit `8902764f`, "6: sparse-hub-base-reconcile")
**Issue:** The declared PATTERN changes list one entry, `PATTERN-sparse-hub-base`, and that entry lands with the declared text.
The same commit also rewrites ten other entries and removes a clause from each:
- `PATTERN-durable-vs-ephemeral-state` loses "nothing a run needs lives only under `.lyx`".
- `PATTERN-hub-wide-config` loses "its machine override beside it".
- `PATTERN-push-both-sides` loses the detached-push clause.
- `PATTERN-driver-choice-single-site` loses "for the surface only".
- `PATTERN-stencil-ownership` loses "never embedded".
- `PATTERN-friction-capture` loses "archived once covered".
- `PATTERN-edit-directive` loses "orch stencils are exempt".
- `PATTERN-testkit` loses "imported only from tests, never duplicated".
- `PATTERN-hubforge-fixtures` loses "never hand-assembled or wrapped".
- `PATTERN-markdown-link-integrity` loses "file part and `#anchor`".

No card asks for these edits and no `Pattern-Departure:` trailer explains them.
Each removal weakens a cross-cutting invariant, and the `pattern/` background files keep the stronger text, so the two now disagree.
The merge base 860c5d03 still carries all ten clauses.

### [LOW] Clone's marker rollback is untested

**Location:** `internal/fabriccli/clone.go:79`
**Issue:** `CloneAndWire` removes the base-sparse marker when `b.Commit` fails, so an untracked marker cannot make the next walk skip its one-time strip.
No test row forces a board-commit failure after the marker write, so that cleanup is unpinned.
This is based on the test names in the diff; the bodies of the clone tests were not opened.

## Rejected

- The code-correctness fork's LOW comment finding on "an open map is compared whole" in `stripBase` and `formerTemplateValues`: the fork marked it uncertain, and `yamlengine.Strip`'s own doc comment uses the same wording, so it is not a defect.
- The code-tests fork's two NIT items on possibly overlapping walk-level and preflight tests: the fork did not read the bodies, and a redundancy finding needs a named covering test and an observed duplicate assertion.
- The code-affected-set fork found nothing.
- Own check of the focus questions:
  - Cards 6 and 8: the strip runs once behind the board marker, an absent marker makes the stamp stale, and a failed step restores its own files and the marker. Held.
  - Cards 4 and 7: sparse write paths keep a seed-only base full with its fill log, and a hub base sparse with no fill log. Held.
  - Card 5: the former-values comparison goes through `yamlengine.Strip` with the module's open maps. Held.
  - Card 9: a failed reconcile warns with its way forward and the start verb goes on. Held.
  - Cards 1 to 3 values and the `orchengine` defaults: as the cards state. Held.
  - Cards 10 and 11: the hub-wide editor uses `BaseScaffold`, and the marker is written in the clone's board commit only for a board seeded from nothing. Held.
- No commit in the range carries a `Pattern-Departure:` trailer.
