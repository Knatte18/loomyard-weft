---
round: 1
focus:
  - "internal/driftengine/judgesnapshot.go and verify.go: does the guard, as the diff leaves it, still halt on a judge-session change to the decision record, while exempting a parent's lym loom decision add or replace made during the judge window? Check the lock handling and the mtime dating of each named change against the issue's two requirements."
  - "internal/driftengine/verify.go resume path: does a resume re-check the guard against the snapshot and name which path still differs and what was restored, rather than repeating the first halt's text? Check failure paths and a stale capture directory."
  - "internal/driftengine/haltfindings.go and verify.go halt reason: does grouping by case, key, kind and reason carry the row range and the first and last old/new values; is the reason capped; does the way forward come first; and is the full findings file written at the repo-relative path the halt text names? Check edge cases: a single row, an empty set, rows that differ only in reason, and a cap boundary."
  - "internal/loomcli/driftwiring.go, deps.go, paths.go, records.go and envkit.go: are the new seams wired in production and in the fixture, so no nil seam reaches a live run? Check the diff against the repository's constraints document and the docs it names for this kind of change."
  - "Scope against the answer key: the decision record's Goal covers issues #146 and #147 only. Does any change in the diff go beyond them, and does either issue's 'What to do' list lack a landed change?"
  - "Change description and commit messages: does each claim in _lym/landing/summary.md, when present, and in the range's commit messages hold against the diff?"
  - "Tests in judgesnapshot_test.go and verify_test.go: does each new test assert the behavior its name states, and does any duplicate another test's coverage? Check that no assertion at the review base was loosened."
---

Seed focus for round 1, set from the decision record's Goal and the diff stat; no prior round exists.
