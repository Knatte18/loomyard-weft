---
round: 1
focus:
  - "internal/battencli/owner.go: does the skip condition cover only the Worktree-Teardown producer with the task worktree absent, so every other resume still runs the reconcile (D1 bound)?"
  - "internal/battencli/owner.go: when the reconcile is skipped, is the owner judged on the local board, and is an entry owned by another machine still refused naming `lym adopt` without the 'once origin answers' re-run hint (D1, D3)?"
  - "internal/battencli/owner.go: does the skipped path write no origin-owed marker (D2), and how does it treat a failure to read the worktree's presence?"
  - "internal/battencli/owner_test.go: does the tier0 table test cover each acceptance criterion over fake seams, without duplicating existing owner or resume tests?"
  - "internal/battencli/resumereconcile.go, docs/overview.md, run_test.go, step_test.go, wire_test.go, lifecycle_integration_test.go: are these edits licensed by the answer key's scope, or are they changes it does not ask for?"
  - "internal/battencli/wire.go: does the failed-deletion error of the Worktree-Teardown row name `lym batten run <slug>` with the slug actually available at that point (D4)?"
---

Seed focus from the decision record's scope and acceptance criteria.
