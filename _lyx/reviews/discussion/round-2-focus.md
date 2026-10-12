---
round: 2
focus:
  - "decision-record.md D10 (orch handoff cleanup) and its Bound: does the file a restart resumes from, which `internal/orchengine/launch.go` takes from `State.LastHandoff` alone and never from a pending handoff, always survive, including after an aborted cycle that left a newer partial handoff file?"
  - "decision-record.md D9 (batten cleanup scratch removal) and its Bound: with `shedrun.RunLock`, `shedrun.StatusLock` and the last-commit lock all living under the slug's scratch directory, does any lock file get deleted while held, and can a concurrent `lyx batten run <slug>`, which reads and seeds `status.json` under the status lock in `internal/battencli/arm.go` without the run lock, interleave with the cleanup's records commit?"
  - "decision-record.md D9 candidate rules: for a slug whose records run directory is gone from disk but still tracked, and for an aged orphan whose records run directory looks absent, what proves the run is done, and what does a stat error other than not-exist do?"
  - "decision-record.md D1, D3 and the Constraints: is the root trust rule (real directory, mode, owner) satisfiable on Windows, where Go reports directory permissions as 0777, and does a root adopted from an inherited `TMPDIR` pass the same checks?"
  - "decision-record.md D4, D6 and the Goal: which test binaries and `t.TempDir` trees are covered when a package has no `TestMain` calling `tmuxkit.Main`, what does a failed run-directory removal do on Windows, and what does each scan allowlist entry let through and is that consistent with the support log's Question ledger?"
  - "decision-record.md D7 and D11: does every refusal and skip of a segment live in the function both its verb and `lyx cleanup` call, and what does the `.scratch` segment do when the board cannot be read?"
---

Focus targets the gating design findings of round 1 and the decisions around them; the reviewer should still read the whole pair.
