```yaml
lens: fix-affected-set
outcome: reported
```

No findings from the set searches. The list below covers the sets I checked and the one I left unchecked.

Checked:
- **F3 (every route by which a guard kill reaches a `GuardKilled` reader):** the only production consumers are `RunAlone` (`internal/testcover/runner.go:187`) and the `startMemoryGuard` closure. Every caller uses the one `onKill` hook. `killTree` is the only place that calls it, so the enclosing-binary pids reach all consumers.
- **F6 (every site that says a guard-killed run is "not judged"):** the matches are `internal/gatecli/doc.go:723-725`, `docs/overview.md:966-970`, `internal/testcover/mutantverdict.go:55` and `internal/testcover/runner.go:69-71`. The two edited sites are corrected. The `testcover` comments describe the redundancy path and are consistent.
- **F7 (every site that states what `memory_mb` bounds):** the sites are `internal/gatecli/doc.go:728-734`, `internal/gateslot/template.yaml:15`, `internal/gatecli/tierrun.go:204` (the cap-line text), `internal/gateslot/doc.go:153` and `docs/overview.md:970-971`. All fit the fallback qualification:
  - `tierrun.go:204`'s cap line prints only on a scope kill, which the fallback never produces.
  - `gateslot/doc.go:153` makes no together-bound claim.
  - `docs/overview.md:970-971` ("runs its whole command under the kernel cap `lym gate redundant` uses") is a summary line with no mention of the fallback.

Unchecked, with the reason:
- **F1, F2, F4:** the premise is one function, not a set of sites. A search for the removed `oomKillPollPeriod`, `markerReadBound`, `scopeCgroup` and `scopeHadOOMKill` outside `memorycap_linux.go` found only a differently named `scopeCgroupPath` in `cmd/testtiming`, which is unrelated.
- **F5:** the claims are in one file. I did not re-read the edited text.

Further tool calls:
1. `grep` for "not judged" and similar phrases across `*.go`, `*.md` and `*.yaml`: finds every site for F6, which the context lacked.
2. `grep` for `memory_mb` sites with bound and kill wording, plus `onKill` and `GuardKilled` callers: finds the F7 and F3 sets, which the context lacked.
3. `grep` of `MemoryMB` and `memory_mb` in `internal/gateslot`, `docs` and `internal/proc/doc.go`: finds further F7 sites the context lacked.
4. `Read` of `internal/gateslot/doc.go` lines 150-157: to check the `memory_mb` clause, whose text the context lacked.
