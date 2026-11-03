---
round: 2
focus:
  - "Settle log and stop markers: `internal/preflight/preflight.go` (`runStateExemptRels` comment) and the `internal/preflight/doc.go` paragraph name a commit that carries the exempt files `go-settle.jsonl`, `stop-marker` and `condition-stop-marker`. Check whether `statusCommitPathspec` in `internal/loomcli/wiring.go` and `battenRunCommitPaths` in `internal/battencli/commitstatus.go` stage those files, or whether the comments name another real carrier, and settle whether the comments and the code agree."
  - "Settle log readers: `internal/arbiterstop/holdreading.go` and `internal/tendercli/deps.go` read the settle log. Check whether, after the Preflight exemption, the log can stay uncommitted while a resume starts, and whether the diff leaves a path that commits it."
  - "Doc currency in `internal/shedrun/paths.go`: check that the `SettleRecords` and `RunsRootRel` doc comments describe every current consumer's use of the root, Preflight's worktree-clean check included."
---

The focus entries are questions for the reviewer to settle, not verdicts.
