```yaml
lens: guard-reach
outcome: reported
```

Lens: guard-reach. I traced each guard, refusal, branch and test row the cards add from a real entry point.

**Findings**

1. **MEDIUM — Card 12, `TestGateTest_DotnetFixture`'s skip when the offline restore finds no test packages.**
   As written, this skip can never fire.
   - The only restore in the card is inside the gate's own `build` command (`dotnet build Fixture.sln -m --source <packages>`). A missing package therefore fails the gate call, and the test fails instead of skipping.
   - To reach the skip, the test would have to restore first, in the fixture worktree, before it calls the gate. That writes untracked `obj/` files (`obj/project.assets.json` and others).
   - Card 6's `DeriveDiff` turns any changed path that is neither inert nor `.cs`, untracked files included, into a whole fallback. So the narrowed-run assertion (`--filter` names `CircleTests` and not `SquareTests`) would fail.
   - Card 6 skips `bin/` and `obj/` only in `scanTree`, not in the changed-path set.
   - Evidence: `internal/testkit/plankit/plankit.go:102`. `plankit.Repo` only writes files under `t.TempDir()`. It runs no `git init` and writes no `.gitignore`, so nothing ignores `bin/`, `obj/` or the worktree's own `.lyx` gate directory, where an outside-hub run puts its records (`internal/gatecli/report.go:89`, `runLogDirOf`).
   - Fix: card 12 names a fixture `.gitignore` for `bin/`, `obj/` and `.lyx/`, plus a restore probe run outside the worktree (or checking the global-packages folder for the pinned versions) as the skip condition.

2. **LOW — Card 10, an inert-only diff under the `none` scoper.**
   - Card 10 builds per-tier shares for `none` through `diffBaseGroups`, the path the `go` branch of `internal/gatecli#computeScope` takes, and that branch returns `guardStep: true` (`internal/gatecli/scope.go:179-184`).
   - If the `none` branch copies that shape, `internal/gatecli#planTierRuns` refuses the call: `--guards needs the go scoper` (`internal/gatecli/tiers.go:117-118`). Every inert-only `none` call would then be refused, not run as empty.
   - The card does not say the `none` inert scope plans no guard step. Its `none` test row reaches this path, so the gap would surface there, but the intent should name it.

3. **LOW — Card 9 against Card 11, a `shared_build` tier with `carry: false` under a `noTests` share.**
   - Card 11 sets `buildOnly` on every `shared_build` tier whose share is `noTests`.
   - Card 9 runs a `NoCarry` tier whole under a `noTests` share, so that tier also has invocations.
   - Neither card says which wins, or that `buildOnly` is set only for a tier left with no invocation. A tier holding both flags reaches `runListed` in a state neither card defines.

4. **LOW — Card 2, the build timeout line at the landing rows.**
   - Card 2 makes `FindTierTimeout` return `build`, so `verifytree` marks a build timeout as timed out.
   - The Merge-In/Finalize timed-out reason (`internal/landingshed/verifygate.go:270-271`) still names, as the way forward, rewriting `tiers.tier<N>={...}`. For a build timeout that is the wrong key; the right one is `build={...}`.
   - No card lists this site, so a build timeout reaches it with a misleading way forward.

5. **LOW — Card 6, `DeriveCards`' fallback for a seed `File` naming a `.cs` file outside the tier's `dir`.**
   - The branch is reachable whenever a tier's `dir` is a subdirectory.
   - No `TestDeriveCards` row reaches it: the listed fallback rows are a non-inert `.json` file, a deleted `.cs` file and an empty seed. `TestDeriveDiff` covers the matching diff branch, but not this card one.

**Further tool calls**
- `grep internal/testkit/plankit` for `Repo` and ignore handling: my context did not show whether `plankit.Repo` creates a git repo or ignores build outputs.
- `sed -n 95,130p internal/testkit/plankit/plankit.go`: to read `Repo`'s body and confirm it writes files only, with no `.gitignore`.
