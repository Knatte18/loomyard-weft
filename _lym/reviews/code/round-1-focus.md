---
round: 1
focus:
  - "internal/shuttleengine/codexengine/command.go and capabilities.go: do the PostToolUse block, the merged Stop block and the idle wake through `codex queue` each deliver claimed mail as the Goal describes, and does a hook failure path (no mail, queue command failing, unreadable thread id) leave the Codex session running instead of blocking or losing mail?"
  - "internal/creelcli/hookwait.go and hookread_test.go: does the change to the waiter keep Claude's existing behaviour intact while serving Codex, and does the idle waiter wake only an idle session, never one mid-turn?"
  - "internal/shuttleengine/creelhooks.go and the claudeengine edits: does the move of the trust sentence, flag directory and flag-name fragment keep every existing Claude consumer's behaviour and output unchanged?"
  - "Goal coverage: is the `[creel]` trust sentence in the Codex prompt file, is the launch marked `CreelHooks`, and is the package doc's 'no creel hooks' removed everywhere it appeared (codexengine doc, README.md, docs/overview.md, shuttleengine and creelengine docs)?"
  - "Test coverage against the Goal: is each delivery path (PostToolUse output, Stop output, daemon wake call) covered by a unit test on hook output or the wake call, and do any new tests duplicate an existing test's coverage or assert only static prose?"
  - "Commit message and landing summary claims against the diff, and the repository's own constraints and doc rules for the files touched."
---

Seed focus drawn from the diff stat and the Goal; the decision record carries no Scope or Acceptance sections, so the Goal and its Wanted list are the measuring stick.
