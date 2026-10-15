---
round: 1
focus:
  - "internal/planglyph/scope.go `symbolFilesCovered`: does the file/package cover logic agree with `targetsCover` and `RefScope`/`RefMember` semantics, including Windows-style paths, `plan:` handles and a nil plan, and the Renamed, Modified and multi-file symbol branches of `ScopeGuard`?"
  - "internal/planglyph/scope_test.go: does the rewritten test table still cover every pre-existing `ScopeGuard` behavior it replaced, and does each new case exercise a distinct coverage path rather than duplicating one (test economy)?"
  - "contracts/stencils/webster/webster-body-implementer.md step 2: does the new deviation rule match card 3's Intent and `ScopeGuard`'s actual counting, and does it drop, weaken or contradict any earlier rule, notably the former step 3 guard-fix bound and the bounded self-fix section?"
  - "internal/planparser `RefMember`: do the card 1 mechanical checks (rename and callers of the old `refMember`) hold, and is the nil-plan default-language behavior tested in the existing test file without a duplicate?"
  - "contracts/specs/loom-plan-spec.md deviation paragraph and `internal/planglyph/doc.go`: do they state the same union rule as the code, with docs landing in the same commit as the behavior change?"
---

Seed focus for round 1, drawn from the four card commits since the parent.
