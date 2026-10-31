---
round: 1
focus:
  - "internal/orchengine/watcher.go, every site that sets CompactionBaseline or starts a reload (tickClearing, startAutoReload, tickCompacting, any resume-after-restart path): does each set PointerMarkSince from the baseline before it moves, and is the value persisted before the pointer step can run?"
  - "internal/orchengine/watcher.go toIdle and state.go ResetForFreshLaunch: is PointerMarkSince zeroed on every return to idle and on a fresh launch, including abort paths that bypass toIdle?"
  - "Decision D1 on the /clear path: the field takes the baseline unchanged; does hookDelivered behave as before there, and does a zero field (older state file) accept any matching-text mark as the record's Bound says?"
  - "Tests in watcher_compact_test.go and watcher_test.go: is each acceptance criterion covered (automatic compaction, requested compact cycle, mark older than previous baseline, other text, field zero after idle), and does any new test duplicate an existing one?"
  - "doc.go and the State field doc: do they describe the new check accurately, and does the diff touch anything outside the scope the record lists (hook verb, ResumeContext, mark file format)?"
---

Seed focus for the diff over the settled range, measured against the board entry and decision record D1 and D2.
