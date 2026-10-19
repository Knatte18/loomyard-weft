---
round: 1
focus:
  - "internal/toolfloor and its doc.go audit: settle whether the floor constants stay git 2.43 and tmux 3.4, the package stays stdlib-only and spawns and logs nothing, and the audit records allow-set-title (tmux 3.5) as an optional above-floor feature with reed's Warn-and-continue fallback."
  - "internal/gitexec, internal/reedengine and the reedcli attach path: settle whether each floor check runs once per process (gitexec) or once per resolved path (reed), caches only parsed answers, retries a probe that could not run, and admits an unparseable line with one Warn."
  - "CherryPick in the gitrepo/fabricengine code: settle whether it no longer uses cherry-pick --empty=drop, skips an already-carried change, and refuses an empty commit, and whether the old flag is gone from every caller and test."
  - "internal/reedengine session row: settle whether the two single-format loops put the marked prime's session first and drop no behaviour on newer tmux."
  - "creelengine deliver tests and the merge-lock tests: settle whether each waits on its probe or lock-wait signal rather than a sleep, and whether any new test duplicates existing coverage (test economy)."
  - "pattern/PATTERN-leaf-packages.md and PATTERN.md: settle whether the new leaf is listed and PATTERN.md gains no entry line, as the no-pattern-entry decision states."
---

Focus for round 1 follows the plan's shared decisions and card index.
Card 1 was committed before this generation, so the review range may not include it.
