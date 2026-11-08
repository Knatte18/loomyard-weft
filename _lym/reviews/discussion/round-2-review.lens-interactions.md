```yaml
lens: interactions
outcome: reported
```

Task: interactions lens. I checked each pair of decisions that act on the same state, then the acceptance criteria and constraints.

**Pairs checked**
- D3 × D4 (block key table in the map): held.
- D4 × D5 (the map's per-file hashes against D5's "tracked at HEAD" stale rule): held. A dirty or untracked package is skipped, so the recorded set equals the tracked set.
- D4 × D5 discard (a whole-record replace while D7 reads the map): held.
- D5 dirty or locked skip × D6 retrigger on packages the cap left: held. A skipped package beyond the cap causes at most one refresh that builds nothing, and that refresh reports nothing left over.
- D5 failure outcomes × D6 retrigger on packages the cap left: **broken** (F2).
- D5 hash discard × D6 "waits for the next prime HEAD change": held. The landing that changed the files moved the prime HEAD past D6's recorded start HEAD.
- D5 time bound × D6 orphan bound: **broken** (F4).
- D6 registration under `HubOwner` × D6 dobby-stop claim: **broken** (F1).
- D6 in-flight record × D6 restart and first-tick start: held.
- D5 prime-only refusal × D6 trigger in the prime parent's worktree: held.
- D1 eligibility move × D7 step 2 and D5 map eligibility (the spawn scan): **broken** (F3).
- D2 module-instrumented tool × D4 and D7 package-scoped coverage: held. These are separate runs.
- D7 repeat skip × D8 match: held. A red report records no hashes, so it never matches.
- D7 step 1 merge base × D8 step 1 on HEAD: held.
- D7 keep drop × D8 keep finding: held.
- D7 report in the gate directory × green record and `verifytree` dirty check: held. The gate directory is never tracked.
- D7 config fill × Constraint `PATTERN-config-layers` (no reinterpretation ships): held.
- D7 "never changes the gate's result" × D7 unknown-report refusal: held. The refusal is a deliberate config refusal.
- D7 step 5 greedy pass × D7 acceptance "reported unique": **broken** (F5).
- D3 ordinal bound × D7 step 5: held.

**Findings**

F1 — BLOCKING, design, D6 (decision-record.md:117–119, acceptance line 235)
D6 registers the refresh child under `runproc.HubOwner` with its own role, `test-map`. That registration defeats D6's claim that a dobby stop ends the child.
- The prime dobby is itself registered under `HubOwner` (`internal/beamcli/dobbydeps.go:77`; beamcli doc line 353, "or runproc.HubOwner for the prime").
- The stop calls `runproc.EndRole(dir, HubOwner, RoleDobby, …)` (`internal/orchcli/cli.go:276`).
- `ender.end` marks every record of that owner with another role as spared, and `skip` drops spared pids from the descendant plan (`internal/runproc/end.go`, the `spared` loop in `end`).
- `EndRole`'s doc says it directly: "Every record of another role, and every process below such a record, is left alone" (`internal/runproc/end.go:72–74`).

So a dobby stop leaves the `test-map` refresh running, and the acceptance criterion "D6: a dobby stop ends a running refresh" cannot be met as designed. Two ways out: the stop also calls `EndRole(HubOwner, RoleTestMap)`, or D6 drops the claim and relies on the orphan bound.

F2 — MEDIUM, design, D5 × D6 (decision-record.md:81–86, 97, 112)
D5 states no outcome for a package whose coverage build fails, or which has nothing to map: no test files, or no `tier0` or `tier1` test. It also does not say whether such a package counts against `--max`.
- Under "it has no map" such a package stays stale forever.
- If it counts against `--max`, then once `test_map_batch` such packages come first in path order, every refresh spends its batch on them. Each refresh then reports packages the cap left unbuilt, and D6 starts a new refresh on every tick. The packages after them never get a map.
- D6's "Packages a refresh skipped never start another" covers only dirty and locked skips.

Fix: such a package either gets a map recording its file hashes, with zero tests or the failure, so that it is not stale until its files change; or it is reported as skipped and does not feed the retrigger on packages the cap left.

F3 — MEDIUM, consistency, D1 × D7 step 2 (decision-record.md:32, 34, 146)
D1 moves the eligibility rules into `internal/testcover`, giving as its reason that a `main` package cannot be imported. Yet D1 and D7 step 2 both still cite the scan as "the static scan in `cmd/testtiming/spawnscan.go`". The gate step lives in `lym gate test`'s package tree and cannot call code in `cmd/testtiming`. D5's map needs the same exclusion, so that an out-of-process test never counts as a cover. Fix: D1 states that the spawn scan moves into the engine, and D7 step 2 cites it there.

F4 — LOW, consistency, D5 bound × D6 bound (decision-record.md:92, 106, 121, 128)
D6 bounds a refresh orphaned by a dead dobby by "D5's time bound". D5's bound counts one `lym` build, the coverage builds and the per-test timeouts. It leaves out the wait to acquire the one gate slot D5 holds, and it leaves out coverage builds, which have no timeout of their own. D6's "lasts no longer than D5's time bound" is therefore not bounded by what D5 states. Fix: add the slot wait and a build timeout to D5's bound, or narrow D6's claim.

F5 — LOW, consistency, D7 acceptance × D7 step 5 (decision-record.md:152, 237)
The criterion reads "a test covering a block of an edited function that no map test covers is reported unique". Under step 5, another judged test covering that same block is also a cover, and the greedy pass can then report the test contained. The criterion contradicts the step whenever two judged tests share the block. Fix: the criterion says "that no map test and no other kept judged test covers".

Tool calls: none; every check used inherited context.
