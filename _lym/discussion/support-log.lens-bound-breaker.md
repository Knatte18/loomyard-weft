```yaml
lens: bound-breaker
outcome: reported
```

Lens: bound-breaker. I attacked the claims and bounds in the decision record and support log.

**Findings**

1. **MEDIUM, D6: the landing trigger misses landings and can be switched off by the board-tidy setting.**
   - `Deps.Landings` (loop.go ~110) counts only "this dobby's runs that hold a Land Done naming this machine", and "a nil Landings, or a nil TidyShuttle, leaves the board tidy off."
   - When the hub wires board tidy off, `countLandings` gives D6 nothing to trigger on.
   - Landings made on another machine, a beam landing that is not a prime-dobby run, and a direct commit to the prime branch never trigger a refresh either.
   - The tidy step also clears `LandedSinceTidy` (tidy.go ~133, 221). "Reusing the landing count" therefore needs its own counter or marker, or a tidy can drop a pending refresh.
   - Fix: D6 keeps its own marker, such as the last refreshed prime HEAD, and does not depend on the tidy wiring. Or it compares the prime HEAD against the commits recorded in the maps on each tick.

2. **MEDIUM, D5 and D7: the staleness test ignores changes outside the package directory, so a map can go wrong and never become stale.**
   - D5 hashes only the Go files in the package directory.
   - These changes alter which of the package's own blocks a test covers, yet never mark the map stale:
     - a change in a dependency or a testkit fixture;
     - a `testdata/` file;
     - a tier's `env` in `gate.yaml`.
   - D7's bound says it "flags a test falsely only when a map test lost coverage since the map was built". With this staleness test, how long that can last has no limit.
   - Fix: also mark a map stale when its age passes a set number of prime commits or a set time, or when a dependency changes. Or the report should state how old the map is.

3. **MEDIUM, D7: two tests judged in the same run can each be the other's only cover.**
   - Counterexample: a diff adds TestX and TestY, which cover the same new blocks. No map test covers those blocks.
   - Step 4 counts "another test judged in this step" as a cover, so the step reports both as contained, each naming the other.
   - A reviewer acting on both findings deletes both, and the new code loses its coverage.
   - So D7's claim that it flags falsely only when a map test lost coverage is broken. It also breaks the PATTERN rule that only a greedy removable set may be deleted together.
   - Fix: apply the greedy pass to the judged tests too, or have the report mark findings that cover each other as "fold one, not both".

4. **MEDIUM, D5 and D7: a prebuilt `lym` and `tier1` tests on keys that run only `tier0`.**
   - The shipped template sets `webster`, `darn-write` and `code-burler-fix` to `tiers: [0]`.
   - On those keys the step is the first and only run of a changed `tier1` test, and no tier run has prebuilt `lym`.
   - D5 says the verb uses "the gate's prebuilt `lym`", and D7 gives the step no source for `lym` at all.
   - The cost claim, "the cost to those tests' own run time", leaves out a `lym` build and a coverage build per package.
   - Fix: say the step builds `lym` once when it runs any `tier1` test, and count that in its cost.

5. **MEDIUM, D7: "never fails the gate" does not hold when a test hangs.**
   - On a `tier0` key, a new `tier1` test runs for the first time in the step.
   - If it deadlocks, the step waits up to the tier's `test_timeout` for each test, and 40 tests are allowed.
   - The gate call's watchdog or the agent's Bash timeout then fails the whole call.
   - Fix: give each test a short timeout of its own and a total budget for the step. A test past either is listed as not judged.

6. **MEDIUM, D8: HEAD can move between the gate and the review.**
   - The marker is filled only when the report's commit equals the reviewed HEAD.
   - The loop owner commits run records under `_lym` between phases. If it commits after Webster's end gate and before Code-Burler-Review, HEAD no longer matches, and the reviewer gets "no current report" every time.
   - Fix: compare the code trees with `_lym/` excluded, or use the newest report whose commit is an ancestor of HEAD and that has no code changes since. I did not confirm the commit order between phases.

7. **MEDIUM, D5, D6 and Open risks: the first refresh is far larger than the record says.**
   - Once this lands, every package on every machine has no map. The first counted landing makes the dobby run `lym gate testmap` over the whole module, serially, in one slot.
   - That is roughly the cost of a full serial `-redundancy` run, not "about four minutes" for one package.
   - The bound "one slot's worth at most" holds for the number of slots, but says nothing about how long the slot stays held.
   - Fix: state the total first-run cost. Or have a refresh call rebuild a set number of packages and leave the rest stale until a later tick.

8. **LOW, D3: the block key fails on methods with the same name.**
   - "Enclosing function or method name" does not tell `(*A).String` from `(*B).String` in one file, and a package-level func literal has no enclosing name.
   - Fix: include the receiver type in the key, with a fixed name for code outside any function.

9. **LOW, D5: "differ from HEAD" is unclear.**
   - It does not say whether an untracked new test file in the package directory counts as differing.
   - If it does not count, the map records HEAD while its coverage includes the untracked test.

10. **LOW, D7: the cap's selection order is unspecified.**
   - Which 40 tests get judged is not stated. Name an order, such as file then name, so repeated runs report the same set.

**Claims attacked**

- D1, the shared engine and eligibility rules kept unchanged: held.
- D2, one module-instrumented run gives both views: held, because a module-wide profile contains every block of the package's own files.
- D2, `-pkg` splicing as today: held.
- D3, the content key survives line shifts: held. Its ordinal collision is admitted. Same-named methods break it (finding 8).
- D4, readers see the whole old map or the whole new one through `internal/state`: held, though I did not verify that the rewrite is atomic.
- D5, the per-package lock skips a concurrent rebuild: held.
- D5, the bound of one slot: held for slot count, broken for duration (finding 7).
- D5, the prebuilt `lym`: broken (finding 4).
- D6, the prime dobby is identifiable: held, since the prime has an empty `Slug` in `Geometry`.
- D6, the prime worktree's tree updates on a landing: held, since fabric's `MergeIn` merges in the checkout.
- D6, the trigger: broken (finding 1).
- D7, the merge base is the one the `card` key uses: held, per PATTERN-test-speed.
- D7, a test that covers changed blocks reads as unique: held.
- D7, a test removed or changed since the map is not counted as a cover: held.
- D7, its only false positive comes from map drift: broken (findings 2 and 3).
- D7, it never fails the gate: broken (finding 5).
- D7, its cost: understated (finding 4).
- D7, a package with no map and a stale map are reported: held.
- D8, the HEAD match: at risk (finding 6).
- Constraint that no test launches a test binary: held; stand-ins are enough.

**Tool calls**

1. `internal/dobbyengine/geometry.go` and the function list of `internal/gatecli/green.go`: to check how the prime dobby is identified and where the gate directory is. My context had only the file names.
2. `internal/dobbyengine/land.go`, `state.go` and the landing lines of `loop.go`: to learn which landings the count includes. My context had only the tidy step's doc lines.
3. A grep of fabricengine for how a landing updates the parent checkout: to test D6's assumption that the prime worktree's tree updates.
4. The doc comments of `internal/fabricengine/mergelifecycle.go`: to confirm a merge into the parent happens in the checkout itself.
