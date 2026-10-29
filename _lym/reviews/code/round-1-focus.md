---
round: 1
focus:
  - "internal/gatecli/run.go and tiers.go: is the not-run line printed only on a passing, non-halted `--card` call, and never under the computed, `--base`, `--all` or explicit-target forms (D1, D2, D3)?"
  - "internal/gatecli/tiers.go: does the not-run set equal the declared tiers minus tiers the call listed or dropped as covered by a listed tier, with manual tiers counted and vet-under-tag runs not counted as running a tier?"
  - "internal/gatecli/run.go: does the line change no exit code, record or green-record entry, and match the decision record's wording and stderr destination?"
  - "internal/gatecli/scopes_integration_test.go: do the added assertions cover the pass with tier1 named, a covered tier not named, another scope form silent and a failing card call silent, within the existing --card subtests and tier?"
  - "internal/gatecli/doc.go and contracts/specs/loom-plan-spec.md: do both state the line, its tiers and its bound, and does the spec's Tier 1 bullet stay consistent with the code?"
---

Scope is a single commit over five files; the decision record's D1 to D3 and acceptance criteria are the measuring stick.
