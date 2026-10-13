---
round: 1
focus:
  - "Where the task's code or stencils (Bouncer rule, `lyx loom circling continue`, orch and arbiter stencils) send text to, interrupt or wait on an agent session, check whether it goes through the parent's new guarded-interrupt and pane-text-injection paths in `internal/shuttleengine` (`interruptguard.go`, `send.go`) and follows the changed `PATTERN-guarded-interrupt.md` and `PATTERN-pane-text-injection.md`."
  - "In `cmd/lyx/testdata/test-budget.yaml`, check that the merged rows hold both sides' new top-level tests: no package the task added tests to is still at a pre-task count, and no row the parent raised was lowered by the merge."
  - "In `docs/overview.md`, check that the merged module table keeps both sides' edits: the parent's reed `--anchor own-window` and shuttle `open` text and the task's hard-cap, escalation-cause and judge-tag text, with no duplicated or concatenated entry."
  - "Settle whether the parent's strand-per-window change in reed (own tmux window per strand, `lyx shuttle open`) alters any assumption the task's escalation, arbiter or orch-guidance text makes about where an escalated or arbiter session runs or is attached."
---

Seed focus from the merge note: overlap is limited to the test budget and `docs/overview.md`, so the entries lean on the changed pattern files and the shared docs.
