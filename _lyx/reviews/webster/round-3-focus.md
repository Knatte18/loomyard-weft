---
round: 3
focus:
  - "Go comments the diff adds or changes, checked against PATTERN-comment-line-breaks: grep the diff's added comment lines for a line that ends mid-phrase or carries a sentence end followed by more text, across every touched file, not only the files already swept; settle whether every such comment is one sentence per line."
  - "New or changed untagged tests in the diff, checked against PATTERN-test-speed: for each, settle whether it calls t.Parallel or carries a comment naming the process-global state (for example the package-level inputSleep in internal/shuttleengine) that keeps it serial, including subtests that have their own fixtures."
---

Focus targets the two areas that failed a fresh review in round 2 and the fixes landed since.
