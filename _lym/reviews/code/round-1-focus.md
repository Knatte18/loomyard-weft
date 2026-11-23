---
round: 1
focus:
  - "internal/preflight/preflight.go runStateExemptRels: does `shedrun.UsageDirRel()` return a path rooted where the other exempt entries are (relative to `_lym`, anchor and subpath cases), so the exemption matches `_lym/usage/session-<id>.json` for both untracked and tracked-then-modified records?"
  - "Scope against the board entry (issues #190 and #194): the diff exempts `_lym/usage`; does that satisfy both reports, and does it exempt anything beyond the usage directory that the entry does not license?"
  - "internal/preflight/preflight_integration_test.go TestCheckResolved_LymOwnStateIsExempt: does the added usage record go through the real writer into the real usage dir, and does the test still pin that a file under `_lym` outside the exempt state refuses?"
  - "Doc comments in preflight.go and doc.go: do the new sentences state what the code does without contradicting the surrounding exempt-state description?"
---

Small diff: one extra entry in the exempt list, doc updates, one added fixture in the existing test.
