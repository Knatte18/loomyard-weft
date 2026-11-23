```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens. For each change in the diff, find everything that reads, asserts or describes it, and flag any of those the diff leaves wrong.

**Per change, the set found:**

1. **`runStateExemptRels` adds `shedrun.UsageDirRel()`** (`internal/preflight/preflight.go:28-35`). Covered.
   - Its only caller is `fabricengine.Clean(l, runStateExemptRels()...)` at `internal/preflight/preflight.go:89`.
   - `Clean` passes the exempt list through `exemptPathspec`, scoped under `anchorRel` (`internal/fabricengine/warpclean.go:28,52-61`).
   - `UsageDirRel` is anchor-relative and rooted at `_lym`, like `RunsRootRel` and `LoomFrictionDirRel` (`internal/shedrun/usage.go:29`, `internal/shedrun/paths.go:475`, `internal/loomengine/config.go:290,358`).
   - The other consumers of the usage directory already include it: `loomRecordsDirs` in `internal/loomcli/driftwiring.go:386`, and the commit paths at `internal/loomcli/sharedbootstrap.go:187` and `internal/loomcli/wiring.go:130`. That matches the comment's claim that a later records commit picks the record up.
   - The consumer of the worktree-clean verdict is `internal/preflightshed/preflight.go:47`, which branches on `CheckWorktreeClean`. Neither the check ID nor the reason format changed, so nothing there is left stale.
2. **The doc comment on `runStateExemptRels`** (`internal/preflight/preflight.go:16-27`) now describes the usage directory. Covered.
3. **The package doc lists the exempt set** (`internal/preflight/doc.go:18-22`) and now includes the usage directory. Covered.
   - No other doc, spec, stencil or PATTERN background lists the exempt set. Searched for: `worktree-clean`, `exempt`, `run-records root`, `friction directory`.
   - `internal/preflightshed/doc.go:12` and `internal/preflightshed/preflight.go:223` mention worktree-clean only about the config reconcile, so they are unaffected.
   - `PATTERN-sole-parsers` mentions the friction directory only as the place a note is written, which this diff doesn't touch.
   - The "Leave the worktree clean" lines in the gatefix, webster and weaver stencils are agent rules, not the exempt set.
4. **Test fixture: `TestCheckResolved_LymOwnStateIsExempt` writes a usage record through `shedrun.WriteSessionUsage`** (`internal/preflight/preflight_integration_test.go:476-479`). Covered.
   - `PATTERN-run-records` (`pattern/PATTERN-run-records.md:21-22`) limits production session-usage writes to shed's one usage writer. A test fixture calling that writer function is not a second writer, and no production code in the diff writes usage records.
5. **Repository rules, declared PATTERN changes, commit messages.** Covered.
   - No PATTERN changes are declared, so none had to land.
   - Neither commit carries a `Pattern-Departure:` or `Loosened-Check:` trailer.
   - The `Root-Cause:` trailer on commit 20977989f names the omission in `runStateExemptRels`, which is the line this diff changes.
   - `PATTERN-code-conventions` holds: the path comes from the `shedrun.UsageDirRel()` accessor, built from `lymdirs` constants, not hard-coded.

**Findings:** none.

**Tool calls:**
1. `grep` for files mentioning worktree-clean: my context had no search for docs, stencils or specs that describe the check.
2. `grep` of those hits and of `pattern/`, `PATTERN.md` and `_lym/specs` for the exempt-set wording: to check whether any of them lists the exempt directories.
3. `grep` of the three stencils' worktree-clean lines and of the usage rule in `pattern/PATTERN-run-records.md`: to confirm the stencil hits and read the usage-writer rule, which my context lacked.
