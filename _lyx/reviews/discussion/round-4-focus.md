---
round: 4
focus:
  - "D8 in _lyx/discussion/decision-record.md and the Scope section: settle whether every test-side watchdog recognizer and /proc pid, argv or cwd probe in the repo (internal/reedcli, internal/loomcli, internal/orchcli smoke and helper test files) ends up on tmuxkit's exported probes under a stated rule, so no test package keeps its own copy, as PATTERN-testkit requires."
  - "D12 in _lyx/discussion/decision-record.md: settle whether the stated rule for which rev-parse spawns in internal/fabricengine survive covers answers git derives from config, such as internal/fabricengine/hook.go's rev-parse --git-path hooks honouring core.hooksPath, and whether the acceptance criteria agree with that rule."
  - "D12's repository test for lyxcwd's .git walk-up: settle whether it matches git rev-parse --show-toplevel for a gitfile or git dir that holds HEAD and a commondir file whose target is missing or lacks objects and refs, and whether an acceptance criterion pins that case."
---

Each entry restates its own sites and question; check them against the decision record as it stands.
