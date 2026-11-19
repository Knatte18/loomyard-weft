---
round: 1
focus:
  - "internal/shuttleengine/gate.go, internal/verifytree/reach.go, internal/gatefixer (WrapRowGate): does the reach turn follow D3 — one failing result per arrival listing a not-yet-listed unreached function, no turn one failure short of the attempt budget, no cold fixer for a reach-note result, and no failure counted for the wrap's re-run?"
  - "internal/testreach (reader.go, index.go, fresh.go, changedfuncs.go): do the reader and index follow D1 and D2 — own-package tests before other packages, worktree map wins only on matching file hashes, a stale or absent index gives the same answer, only the prime's --map run writes the index, moved or build-constrained functions matched as D2 states?"
  - "internal/testcover/linemutant.go and mutant.go with internal/gatecli mutate: does the planner follow D9 — one mutant per changed line in operator order, none on strings, embedded lines or logger calls, uncovered and not-judged blocks reported, timeout and memory cap unjudged, mutation_max_mutants honoured — and does lym gate mutate and --base follow D11, including the board note it must file?"
  - "internal/gateslot/triage.go, failurereport and fixer-report changes: do triage lines follow D7 and D8 — first four-space line under the test, dropped before any test output is cut, no-change-set wording and no command without a parent branch (the prime too), FailedTests parsing unchanged, refusals with a way forward?"
  - "internal/shedadapters/rubric.go, shedrecipe, loomcli/reachnote.go and the pattern/ diff: does the reach_report marker follow D5 (matches only when HEAD's changed functions, hashes and package Go files agree; reasons for no report), and does the PATTERN-coverage-map-reader and PATTERN-test-speed diff land the declared changes?"
  - "Scope against the record: D10 (the mutation report step and its config acceptance) must not be built in this diff; check that D12's template defaults and config keys land and nothing beyond them."
---

Seeded from the decision record's D1–D12 and the diff stat; each entry names where to look and the question to settle.
