---
round: 1
focus:
  - "contracts/stencils/orch/orch-template-role.md: read the card 2 diff against its Intent; does it drop, weaken or contradict any rule the card did not ask to change?"
  - "contracts/stencils/bouncer/bouncer-template-escalation.md and the decide-capped/decide-open fragments: does the brief settle only on a recorded operator decision and decide nothing at cause `budget`, as card 3 asks?"
  - "internal/orchengine/prompt.go and internal/orchcli/start.go, paths.go: is `ORCHESTRATOR.md` read from the told path and filled into the second marker at every role-file render, including resume and watcher paths; what happens when the file is absent?"
  - "internal/orchengine/prompt_test.go, start_test.go, resumecontext_test.go, watcher_test.go: does each new test cover behavior not already covered, and is any assertion on stencil static prose?"
  - "CLAUDE.md and ORCHESTRATOR.md: do the moved rules arrive intact, is the pointer line in place, and does `cmd/lyx/testdata/test-budget.yaml` match the new top-level tests?"
  - "Docs in the same commit: do `internal/orchcli/doc.go` and `internal/orchengine/doc.go` describe the new behavior, and does any PATTERN or docs/overview.md statement need a matching change?"
---

Seed focus from the plan's four cards and the commits that implement them; the base diff review still applies in full.
