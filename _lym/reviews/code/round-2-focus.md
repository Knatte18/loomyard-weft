---
round: 2
focus:
  - "internal/proc/memorycap_linux.go, runMemoryCapped: when the kernel's OOM victim is a child the capped command started rather than the command, with -p OOMPolicy=continue, are the scope's remaining processes still ended before the call returns, and does `killed` stay true only for the command's own SIGKILL?"
  - "internal/proc/memorycap_linux_test.go: does a test fail if -p OOMPolicy=continue is dropped from the systemd-run arguments, and does every poll or wait in the capped-run tests use a generous deadline rather than a short measured-time bound?"
  - "internal/proc/memorycap_linux.go, scopeStartScript and killScope: does the start script name the cgroup listing path only through the package's existing constant and run no PATH-resolved external command before exec, and does killScope open cgroup.kill without creating it, so a kernel lacking it adds no error to a cap kill?"
---

Focus covers the areas the round-1 fixes will rewrite, so a fresh reviewer sees the new text.
