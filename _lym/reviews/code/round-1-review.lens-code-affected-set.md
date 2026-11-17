```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens, covering what the diff's changes force outside the lines it touches.

**Per change, the set found and whether it is covered:**

1. **`memoryScopeArgs` gains `-p OOMPolicy=continue`.**
   - `probeMemoryScope` (`memorycap_linux.go:76`) gets the policy too. That is harmless, since the probe's scope only runs `cat`. Covered.
   - Docs at `doc.go:55-56`, `memorycap.go:11` and the `memoryScopeArgs` doc comment all state the policy. Covered.
   - No test asserts the arguments. No fixture or PATTERN file names `MemoryMax` or `OOMPolicy` outside `internal/proc`. Covered.
2. **`scopeStartScript` now writes the cgroup listing, and `scopeStarted` became `scopeCgroup`.**
   - `scopeStarted` had one caller, `runMemoryCapped`, which is updated. No other reference is left.
   - `markerReadBound`'s doc is updated.
   - `scopeStartScript` repeats the literal `/proc/self/cgroup` even though `selfCgroupFile` already names it. **Not covered: see finding 1.**
3. **`runMemoryCapped` calls `killScope` after a SIGKILL and joins any kill error into `err`.**
   - `internal/testcover/runner.go:178-179` computes `capped := ... err != nil && (killed || ...)`. A joined error is still non-nil and `killed` stays true, so the result is unchanged.
   - The `runner.go:245` block-table caller treats any `err` as a failure, as it did before.
   - `gatecli/redundantmutants.go:508` reads only `MemoryCapCoversTree`, which this diff does not change. Covered.
   - The spawn-observability exemption for `memorycap_linux.go` (`cmd/lym/spawnobservability_test.go:65-68`) still holds: `killScope` writes a file and starts no process. Covered.
4. **New `cgroupKillFile` constant and `killScope`.** They are named once and used once. The cgroup root reuses `cgroupMount` (`tasklimit.go:17`). Covered.
5. **Docs.**
   - The headers of `memorycap_linux.go` and `memorycap_linux_test.go`, the `RunMemoryCapped` doc and the `proc` package doc are all updated.
   - `testcover/doc.go:16-19` and `gatecli/doc.go:1160` describe only the kill report and tree coverage, which this diff does not change. Covered.
6. **Repository rules, stencils, declared PATTERN changes, commit messages.**
   - The diff changes no stencil.
   - The directive declares no PATTERN change.
   - Neither commit carries a `Pattern-Departure:` trailer.
   - The commit message's claims match the diff: the policy, the start script reporting the cgroup, and the `cgroup.kill` cleanup.
   - The `PATTERN-code-conventions` named-constants rule is checked under finding 1.

**Findings:**

1. **BLOCKING · design · `internal/proc/memorycap_linux.go:47`.** `scopeStartScript` hard-codes `/proc/self/cgroup`, a second literal of a path `selfCgroupFile` (`internal/proc/tasklimit.go:16`) already declares. The `PATTERN-code-conventions` rule "Named constants" (`pattern/PATTERN-code-conventions.md:11`) says such a path is "one named constant, declared once and referenced everywhere, never a repeated literal". The fix is to build the script from the constant, for example `"cat " + selfCgroupFile + ` >&3; exec 3>&-; exec "$@"``.

**Tool calls:**
- A grep of the repository for `systemd-run`, `MemoryMax`, `oom-kill`, `OOMPolicy`, `start line`, `scopeStarted`, `cgroup.kill` and `RunMemoryCapped` across Go, markdown, YAML and template files. The manifest held no search for docs, fixtures or rules outside `internal/proc` that describe the scope.
- A grep of the `killed`/`err` consumers in `testcover` and `gatecli`, the `/proc/self/cgroup` literal and its constant, and the spawn-observability exemption. The manifest had the callers' signatures but not how `runner.go:179` reads `killed` together with `err`, and not the exemption text.
