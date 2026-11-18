---
round: 3
focus:
  - "internal/proc/memorycap_linux_test.go: does every test the redundancy report lists as kept carry a `//testtiming:keep <reason>` line directly above its `func Test…` line, with a reason naming what it pins?"
  - "internal/proc/memorycap_linux_test.go and internal/proc/export_linux_test.go: does any test repeat a path literal such as the cgroup file or cgroup mount that internal/proc/tasklimit.go declares once as a constant, instead of reaching it through an exported test alias?"
---

The two entries follow from the commits that marked the kept tests and swapped the literals for proc's constants.
