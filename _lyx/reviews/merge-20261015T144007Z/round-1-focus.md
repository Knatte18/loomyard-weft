---
round: 1
focus:
  - "internal/agentname: the parent added CallerLabel and CallerOperator, the task added FormatProgram, ParseProgram, IsProgram and External. Does CallerLabel, fed a program sender such as ly:EXTERNAL or ly:BATTEN, return a label that is sensible, and does any creel code path that feeds a sender name to CallerLabel or to Parse assume only strand names?"
  - "internal/agentname and PATTERN-agent-name: do the parent's CallerLabel doc comment and the task's program-name rules (upper-case second segment, Parse refusing a program name) stay consistent with each other and with the task's PATTERN-agent-name edit?"
  - "internal/reedengine/lifecycle.go and its test: the parent added a third chained set-option (@lyx_follow) to serverSpawnArgv and the PanePID field on StrandStatus; the task's creelcli adapter reads reed's strand listing and pane process. Does the merged adapter still match the merged StrandStatus shape and the merged argv, with no stale assumption about the two-pin tail?"
  - "pattern/PATTERN-leaf-packages.md: the parent added leaf entries for durfmt and sleepjournal; the task added packages (creelengine, creelcli) and may have added or edited leaf entries. Does the merged file hold both sides' entries without a lost or duplicated section, and do the task's new imports respect the parent's new leaf rules?"
  - "pattern/PATTERN-pane-binary-resolution.md: the parent widened the key-binding clause to cover the status bar's follow button. Does any task code that composes a pane command or key binding for creel follow that clause?"
  - "cmd/lyx/testdata/test-budget.yaml and docs/overview.md: both sides edited them. Does the merged budget still carry each side's rows with the right packages, and does the merged module table and execution stack list the parent's new modules and creel without one overwriting the other?"
  - "Test economy: do the merged agentname tests (TestCallerLabel from the parent, the program-name cases from the task) or the reedengine lifecycle tests overlap in a way that makes one redundant, and is any behavior of the interaction between CallerLabel and program senders left without a test?"
---

Seeded from the overlap list and the two sides' diffs of `internal/agentname` and `internal/reedengine/lifecycle.go`.
The parent's range touches many unrelated packages; the focus stays on the six overlapping files and the two changed pattern files.
