---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
---

# Round 1 fixer report

## Changes

Commit `bfa50a6f1` fixes the review's findings.
Commit `893dffa92` fixes what the self-check found.

- F1: `continuesItem` ends a bullet at a `#` line only when it is an ATX heading,
  and at a `<` line only when it opens an HTML block of CommonMark's start conditions 1 to 6.
  Added the two proposed rows and a block-tag row.
  The doc comments of `continuesItem` and `SectionBullets`, the spec and the summary state the same rule.
- F2: added the comment and blockquote rows to `TestSectionBullets`.
- F3: rewrote the summary's Merge danger section to name the three readers and the effect on an in-flight raddle proposal.
  The Blast radius line must be one word, so it reads `records` and the names are in the prose below it.
  The door stays two-way: no stored record is rewritten, and refiling a proposal clears the disagreement.
  The summary is under `_lym` and is not committed by this round.
- F4: `continuesItem` ends a bullet at a thematic break; added the proposed row.
- F5: the splice fixture's first criterion runs onto an unindented line, and `part.Criteria` carries it.
- F6: moved the bullet-extent sentences in the discussion spec below the "So `D1` is not met by `D10`" sentence.
- F7: extended the doc comment of `TestSectionBullets`.

A later gate found wrapped comment lines in the doc comments of `continuesItem` and `TestSectionBullets`.
Both are rewritten as one sentence per line, with no code change, in a separate commit.

`lym gate test ./internal/discussionparser ./internal/raddleparser` passed after each commit's changes, and `lym loom validate-description` passed on the summary.

## Self-check

- fix-code-correctness, MEDIUM (spec says an indented line always continues a bullet): fixed as proposed.
  The spec now names a fenced line as an ending.
- fix-code-correctness, LOW (spaced `* * *` and `- - -` read as bullets): fixed with own change.
  `isBulletLine` returns false for a thematic break, and a row pins it.
- fix-consistency, LOW (thematic-break rule stated more widely than the code applies it): fixed, merged with the spaced-break finding.
- fix-consistency, LOW (spec lead sentence omits terminators): fixed, merged with the spec finding.
- fix-proposal, MEDIUM (spec's indented-line claim is false): fixed, merged with the spec finding.
- fix-affected-set, LOW (new exclusion branches have no test row): fixed as proposed.
  Added rows for the raw-text tag, processing instruction, declaration, CDATA and `___` branches.
- fix-affected-set, LOW (spaced thematic breaks never reach the check): fixed, merged with the spaced-break finding.

All four forks returned a report.

## Disputed

## Deferred

## Rulings
