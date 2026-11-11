```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I checked every pair of the record's decisions that act on the same state, then each acceptance criterion against the decisions it touches.

**Pairs checked**
- D1 plan × D5 failed build, for both the affected-tests and the miss outcome: held.
- D1 Current × D4 outcome-entry drop: **broken** (F3).
- D1 plan × D9 hand call on the same package: held through the build lock. Whether the plan skips or waits on a held lock is unstated (LOW, not filed).
- D1 × `testmap-cross-package`'s dirty reads: held, since the plan reads committed content through `CodeFileAt`.
- D2 test-file and own-file rules × D1 outcomes: held.
- D2 failure rule × D5: held.
- D2 stamp miss × D6: **weak** (F6).
- D3 function sets × D2 selector: held.
- D3 × D7 self-check: held, because the fresh profile still lists linked blocks with their enclosing function.
- D3 × the redundancy step's own-block judging: held.
- D4 spawn skip × D9 hand call × D13 `redundancy-sweep-existing`: held.
- D4 outcome entries × D2's "no recorded test" × `gate-scope-coverage` D3: held.
- D5 × D6, a failed build after a toolchain bump: held.
- D5 × D8 trigger: **broken** (F2).
- D7 audit × D8 process end: **broken** (F1).
- D7 key retirement × D1's interim rule: held.
- D8 HEAD check × D9 slot wait: **weak** (F5).
- D8/D12 exit record × D9 hand call: **broken** (F4).
- D8 × D11 anchor move: held, since the dobby starts nothing while a move stands and D10's signal path writes finished rounds.
- D9 × D10, parallel runs within one slot: held.
- D9 hand-named first × D10 order: held.
- D10 discard × D8's HEAD read before each job: held.
- D10 per-round writes × D7 self-check re-measure: held.
- D13's ordering split × D6, D10, D12: **weak** (F7).
- Acceptance criteria: the D1 first criterion and the D7 audit criterion conflict with other decisions (F3, F1). The rest are consistent.
- Constraints: the config-layers constraint × D7/D8 key retirement held; daemon-wakeups × D9 held.

**Findings**

1. **BLOCKING — D7 × D8: the audit has no time to run.**
   - D7 runs the audit "when its queue is empty and it holds a slot".
   - D8 says the upkeep process "ends when its queue is empty".
   - D8 starts a process only when HEAD moved or work is queued, so with HEAD unchanged and the queue empty nothing starts.
   - As written, the audit, which D7 sets up as the replacement for the age limit, never runs, and the D7 audit criterion can't be met.
   - Fix: the process runs one bounded audit round before it ends each pass, or audit-due counts as work in D8's trigger, with its bound stated.

2. **MEDIUM — D5 × D8: a failed build can restart upkeep on every dobby tick.**
   - D5 skips a failed package at the same HEAD.
   - D8 restarts upkeep while the exit record "names queued work".
   - At an unchanged HEAD the plan sees the miss again and queues the rebuild, which D5 then skips.
   - If that package counts as queued, the dobby restarts on every tick. D8 excludes dirty-skipped packages from the queued count, but not failed builds.
   - Fix: D8 also excludes a package whose failure is recorded at the current HEAD.

3. **MEDIUM — D1 Current × D4 outcome-entry drop: the two rules disagree on what "current" keeps.**
   - D1's Current outcome keeps "every entry" and "builds and runs nothing".
   - D4 drops an outcome entry whenever any recorded file differs between `Commit` and HEAD, and D1 queues every declared test with no entry.
   - So any body change to an uncovered linked function drops every outcome entry and queues a build. The map is then not Current, which breaks D1's first criterion for any map holding an outcome entry.
   - It also makes failing tests rebuild their packages after nearly every landing that touches a linked file.
   - Fix: either the plan treats outcome entries apart from the Current verdict, or the rule narrows to the package's own files plus D2's miss cases.

4. **MEDIUM — D9 hand call × D8/D12 exit record: a hand call can stall the backlog.**
   - Today `close` writes the exit record on every call, hand calls included.
   - Under D12, a hand call would overwrite the planned HEAD and the queued count with its own.
   - The dobby then reads "HEAD planned, nothing queued" and the backlog stalls until the next landing. The record doesn't say who writes the exit record.
   - Fix: only the dobby-started upkeep process writes the exit record.

5. **LOW — D8 × D9: a process waiting for a slot ignores HEAD moves.**
   - D8 checks HEAD only before each package job.
   - A process waiting under D9 stays live across landings, and the dobby won't start a fresh plan while it lives.
   - Re-stamps for newer HEADs wait until it gets a slot.
   - Fix: also check HEAD during D9's wait.

6. **LOW — D6: a tuned `parallel` value rebuilds every map.**
   - The stamp digests the tiers' rendered environment.
   - `renderTierEnv` in `internal/gatecli/tiers.go` substitutes `{parallel}` with the tier's parallel value.
   - Changing a tier's `parallel` therefore changes every stamp: a whole-module miss and rebuild for a setting that doesn't change coverage.
   - Fix: digest the env before the `{parallel}` substitution, or state this as accepted.

7. **LOW — D13: the ordering split is incomplete.**
   - D13 puts D10 in the after-the-selector group, but D10 (parallel runs, per-round writes, order) calls no selector.
   - It doesn't place D6 or D12 in either group.

Tool calls:
- grep of `renderTierEnv` in `internal/gatecli/tiers.go`, to check whether D6's environment digest depends on the worktree or on config values. My context didn't hold its body.
