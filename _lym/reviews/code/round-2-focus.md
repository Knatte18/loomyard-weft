---
round: 2
focus:
  - "internal/proc/memorylimit.go and internal/proc/memorycap_linux.go: is the cgroup `memory.max` file name declared once as a constant on a platform-neutral file and used by every reader, with no repeated literal left in the package?"
  - "internal/proc/memorycap_linux.go `scopeStartScript` and the `runMemoryCapped` release path: does the scope holder start in its own session, and does a test pin that the holder ends once a capped run with no kill returns?"
  - "internal/proc/memorycap_linux_test.go: do the new scope-only tests skip where `MemoryCapCoversTree` is false, assert the holder is no child of the command, and exercise a cap that is not a page multiple?"
  - "internal/gatecli/tierrun_memorycap_linux_test.go: is the hog in `TestRunTierMemoryCapKillsARunawayCommandAndNamesTheCap` bounded to a fixed size above the cap, and do its comments describe only behavior the code still has?"
  - "_lym/landing/summary.md: does every claim about repeated gate runs name run ids and overlapping spans that the gate run records bear out?"
---
