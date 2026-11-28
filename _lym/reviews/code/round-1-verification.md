---
round: 1
entries:
  - key: reset-counts-lym-own-ignored-files-as-work
    status: closed
    finding: F1
    reason: "3012812e5: ignoredWork skips what lym writes (lymWrittenInPrime, Bolt locks and manifest) through constants, with doc comment, doc.go (also 59fd88796) and test rows; not done: _lym/landing/summary.md still says every ignored path counts as work, a run record the fixer does not write"
---
