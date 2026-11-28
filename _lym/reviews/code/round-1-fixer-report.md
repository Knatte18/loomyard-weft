---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed-differently
    reason: "The proposed stop path is in both fragments, with one addition the fix key lacks: with no parent, calls an `Added after Discussion` entry allows are run. The proposed unconditional stop would repeat on every resume, since the stop's way forward records such an entry (internal/loomshed/darnwrite.go:32) and a run with no parent stays so. The stop also ends with no further commit rather than no commit, because a mixed entry may already hold code commits."
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
---

# Fixer report, code round 1

## Changes

- F1: removed the `measurementEntryRule` constant and its two uses from `internal/darnengine/spec_test.go`. The commit carries a `Loosened-Check:` trailer.
- F2 (fixed differently, see the frontmatter): both Step 4 fragments now stop a writer that has no parent, or whose post is refused with no strand answering. It writes a stop report naming the measurements, creates Step 3's stop marker and commits nothing further, unless an `Added after Discussion` entry allows the calls.
- F3: the ungated fragment keeps the measurement calls in the writer's session, in item 2, and says in item 3 that no subagent runs them.
- F4: both fragments scope the rule to the measurement calls. The ungated one keeps the per-commit gate run, and the gated one keeps the verify gate's test of the committed tree.
- Self-check: `stencils.go`, the `DarnTable` doc and both fragment headers now say "measurement calls". Calls allowed by a record entry run one at a time in the foreground.

F2, F3 and F4 share the fragment files and are one commit. The self-check changes are a further commit.

`lym gate test ./internal/darnengine ./contracts/stencils` passed after the findings' fixes and again after the self-check changes (tier0).

## Self-check

Fork findings, with S1 and S2 as the ids the self-check commit names:

- fix-code-correctness: no findings.
- fix-proposal: no findings.
- fix-affected-set, LOW: the rule's descriptions still said "gate calls" (S1). Fix as proposed.
- fix-consistency, LOW: the same defect (S1), merged with the one above. Fix as proposed.
- fix-consistency, LOW: the record-entry path lacked "one at a time in the foreground" (S2). Fix as proposed.

## Disputed

## Deferred

## Rulings
