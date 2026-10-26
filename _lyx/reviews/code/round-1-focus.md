---
round: 1
focus:
  - "internal/reedengine/proctree_linux.go and proctree.go: does the AT_CLKTCK read keep paneProcessAge's result equal to the sysconf(_SC_CLK_TCK) one, including the fallback to 100 when unix.Auxv errors or returns no entry, and is the key value 17 right for every Linux architecture the module builds for?"
  - "cmd/lyx/nocgo_test.go: with scankit.Options{} and Filter = scankit.All, does the walk cover every Go file of the module (nested directories, tools/, _test files, build-tagged files) and skip nothing a cgo import could hide in? Does the guard follow the repo's //lyx:guard and tier0 conventions in PATTERN-test-speed?"
  - "nocgo_test.go fixture tests: do they cover behavior the guard adds without duplicating the real-tree guard, and do they check the decision record's acceptance criteria (fixture importing \"C\" fails, comment or string mention passes)?"
  - "pattern/PATTERN-gate-slot.md and internal/gateslot/doc.go: does the no-cgo rule land beside the -trimpath bullet with the overview line unchanged, as the decision record's notes ask, and does the commit message's root-cause claim match the doc text?"
  - "Scope: any change beyond removing the cgo use, adding the guard and stating the rule, such as gate.yaml, per-worktree caches or retry in gatecli, which the decision record puts out of scope."
---

Seed focus from the board entry, the decision record and the diff of two commits over six files.
