---
round: 2
focus:
  - "Go comments the committed diff adds or reflows, in every touched file: does each follow semantic line breaks (one sentence per line, a break at an internal independent-clause boundary in a long sentence, no column limit), as PATTERN-comment-line-breaks requires? Check all touched files, not only the first few."
  - "internal/orchengine/state.go `saveStateForStrand`, reached from `Watcher.save`, with `recordStoppingOnCancel` in watchloop.go writing `WatcherStopping`: when a signal arrives mid-tick and the tick then saves the state it loaded before the signal, does the persisted `WatcherStopping` survive, and does a test drive a tick through a save to prove it?"
  - "contracts/stencils/orch/orch-template-reload.md and orch-template-role.md header comments: after an auto-compaction the orch sequence is `/reload-plugins` then the reload pointer with no skills step, and only `/clear` reloads skills; does each comment describe that sequence accurately?"
---

Focus entries name where to look; the full diff is still the subject.
