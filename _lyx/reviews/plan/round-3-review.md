---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: _lyx/plan/10-board-model-docs.md (D13 paragraph)
    summary: The doc.go hand-off paragraph is action-shaped but never asks that the exact list be quoted verbatim in the PR description, so a Describe session may summarise it and drop the list that D13 requires there.
---

### [LOW] doc.go hand-off paragraph does not ask for the verbatim list

**Location:** `_lyx/plan/10-board-model-docs.md`, the D13 paragraphs.

**Issue:**
The card correctly sees that the Describe session reads the decision record, the board entry, the diff and the run records, never commit messages, and so puts the snippet in a `doc.go` paragraph phrased as a pre-deploy hand step.
It states why that phrasing is chosen.
It does not say that the paragraph must tell the reader to reproduce the list verbatim in the PR description.
An action-shaped paragraph can be carried into the PR body as "write the area labels into `board.yaml`" with the list left out, which fails the acceptance criterion that the PR description carries the exact `labels` list.

**Fix:**
Have the card require one sentence in the paragraph asking that the PR description quote the snippet verbatim, and say why: the hub copies the list from the PR description, not from the diff.

## Focus departures

None.
Verdicts on the other two focus questions, neither a defect:

- `04-board-load-migration.md` names D1 as the licence for loading an entry with neither `tier` nor `kind` as a note with empty labels, and states that D3 refuses its next write until it is labelled.
- The `ImpactSummary` of `10-board-model-docs.md` covers the package-doc move, the other doc edits (as "documentation only") and the temporary `doc.go` hand-off paragraph the hub removes.
