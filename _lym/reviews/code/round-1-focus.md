---
round: 1
focus:
  - "internal/lymtemp/run.go, internal/testkit/tmuxkit/tmuxkit.go: does the shortened run name bring the longest socket path for a ten-digit uid under /dev/shm to within maxSocketPath, and is the comparison in checkSocketPath inclusive or exclusive at 107 bytes? Count the bytes from the code, not from the commit message."
  - "Anything else that reads, matches or documents the old `r` plus six hex digits name (the lymtemp sweep, other tests, other pattern/ or docs/ text, specs): does any consumer still assume hex, or six characters?"
  - "internal/lymtemp/run.go randomRunName and the retry bound: is the name space of four characters still enough for runNameAttempts against a root holding many live and stale run directories, and does a taken-name failure stay an error naming the root?"
  - "internal/testkit/tmuxkit/tmuxkit_test.go TestCheckSocketPath: the new case builds its directory from lymtemp.Dir(), so does it test the ten-digit uid under a /dev/shm-length root deterministically, and does it duplicate the existing widest-uid case?"
  - "Scope against the decision record's Part 1: a path valid before must stay valid, other temp-root users keep working, and the choice of cut is justified. Is the PATTERN-lym-temp-root.md and lymtemp doc change the whole of what the repository's rules require for a changed run-name format?"
  - "docs/benchmarks/test-suite-timing.md new section: does each claim the section makes (revision, pair ratios, pass counts, memory peak, the Concurrent column, the Answer bullets) follow from its own table, and does it cover what Part 2 asks for? The section is a measurement report and its numbers are not re-run."
---

Seed focus from the diff stat, commit messages and decision record.
