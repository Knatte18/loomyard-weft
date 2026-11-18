```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. For each changed behaviour, I list what reads or asserts it and whether the diff keeps that consistent.

**Per change, the affected set**

1. **`Config.Gate`: a tierless hub now gets the key, still refused for a missing key or empty list** (`internal/gateslot/tiers.go:363-376`).
   - Covered:
     - `renderGateCall`, `GateCall`, `AgentGateCall` and `TierList` (`tiers.go:385-490`).
     - The callers in `loomcli/roundgate.go:20,31`, `loomcli/gatefixer.go:103`, `runplan/cardgate.go`, `weaverengine/strand.go`, `websterengine` and `webstercli/beginbatch.go:76`. These callers only check the error.
     - `landingshed/verifygate.go:117-122,405-409`, which handles an empty list.
     - `gatecli.gateKeyRefusal` (`run.go:402-414`).
     - `gateslot/doc.go`. Tests `config_test.go:948` and `tiers_test.go:499` still hold, since tiers are declared there.
     - The integration row "a hub with no tier names lym config gate" (`test_integration_test.go:2108-2115`). It passes no `--gate`, so the tier0 refusal still comes from `planTierRuns` (`gatecli/tiers.go:91`).
   - Not covered: the comment in `internal/gateslot/template.yaml:28` says "a tier a key names must be declared above when the gate runs". It is now false for a hub that declares no tier. That is the template's own state, and it is what new hubs read.
2. **`runTest` returns 0 early for `--gate` in a tierless hub** (`run.go:83-86`).
   - Covered: the `gatecli/doc.go` paragraph.
   - Not covered: the `template.yaml` comments at lines 23-24 say a key's lints and reports run after its tiers. The early return skips the template's own `card` key lints `[comments, pattern]` (`template.yaml:28`), and no doc says so. The code-correctness lens owns that defect; this lens reports only that no doc describes it.
3. **`allScope`: a `--gate` key of scope `all` over an inert-only diff from the parent's merge base gets the empty scope** (`scope.go:327-347`).
   - Covered:
     - The `computeScope` doc and the `gatecli/doc.go` scoper paragraph.
     - `docs/overview.md:1000` ("`inert_paths` makes a diff of files no test reads run no tier"), which now reads truer.
     - `docs/dotnet-hub.md:106` ("A diff whose changed paths are all inert … runs no listed tier's command").
     - Test `scopes_integration_test.go:618`, whose change is non-inert (`lib/lib.go`) and stays valid.
     - The explicit `--all` carry tests at `scopes_integration_test.go:731-907` are untouched.
   - Not covered:
     - The `lym gate test` help (`internal/gatecli/cli.go:73`) still says "scope all runs every tier whole".
     - `template.yaml:21` still says "(scope all, the whole tree)".
     - `docs/dotnet-hub.md:131` says "Parent-Align is then the one planned whole-solution run per task", which no longer holds for an inert-only branch.
4. **The darn gated fragment's new exception** (`contracts/stencils/darn/darn-fragment-make-change-gated.md:9-13`).
   - Covered: the step's own rule, the subagents' "run no `lym gate test`" rule (step 2, unchanged and consistent), and PATTERN-code-conventions (no language hard-coded).
   - Not covered:
     - The fragment's header comment (lines 1-2) still says "the writer and its subagents run no gate".
     - The gate's own no-test line (`internal/gatecli/tierrun.go:486`) tells the reader to run the skipped tests by hand with `dotnet test &lt;project&gt; -p:&lt;property&gt;=true` when `refuse_raw` allows it. The new fragment text says "Never run a raw test command for it … name them to your parent". The darn writer therefore gets two contradictory instructions for the same report.
5. **Commit messages**: no `Pattern-Departure:` trailers, and no declared PATTERN changes. The commit bodies match the diff. Nothing to flag.

**Findings**

1. **MEDIUM, consistency.** `contracts/stencils/darn/darn-fragment-make-change-gated.md:12-13`. The new exception says never to run a raw test command and to name a no-test project to the parent. The gate's no-test line, which that exception tells the writer to read, says to run those tests by hand with `dotnet test &lt;project&gt; -p:&lt;property&gt;=true`. Evidence: `internal/gatecli/tierrun.go:486`, and fragment lines 12-13.
2. **LOW, consistency.** `internal/gatecli/cli.go:73`. The `lym gate test` help still says "scope all runs every tier whole", but a `--gate` key of scope `all` over an inert-only diff now runs no tier. Evidence: `cli.go:73` against `scope.go:342-346`.
3. **LOW, consistency.** `internal/gateslot/template.yaml:21,28`. The template that seeds every hub's `gate.yaml` has two stale comments:
   - Scope `all` is still described as "the whole tree".
   - "A tier a key names must be declared above when the gate runs" is no longer true for a hub that declares no tier, which is the template's own `tiers: {}` state.

   Evidence: `template.yaml:18,21,28` against `tiers.go:371-373`.
4. **LOW, consistency.** `contracts/stencils/darn/darn-fragment-make-change-gated.md:1-2`. The header comment still says "the writer and its subagents run no gate", but step 3 now names an exception for the writer. Evidence: fragment lines 1-2 and 9-11.
5. **NIT, consistency.** `docs/dotnet-hub.md:131`. "Parent-Align is then the one planned whole-solution run per task" no longer holds, since an inert-only branch's Parent-Align and Finalize gates run no tier. Evidence: `docs/dotnet-hub.md:123-131` against `scope.go:327-346`.

**Tool calls**
1. Grepped for the tier0 refusal text and the "scope all … whole" phrase repo-wide: the manifest had no list of the docs and tests that assert them.
2. Read `test_integration_test.go:2100-2125`, `tiers_test.go:490-505` and `scopes_integration_test.go:605-630`, and grepped the specs and docs for `inert_paths` and scope `all`: these were the hits from call 1, which the manifest lacked.
3. Read `docs/overview.md:994-1006`, `docs/dotnet-hub.md:95-130` and `template.yaml:8-30`: these docs and the template comments describe the changed behaviours and were not in the manifest.
4. Grepped `gatecli` for role-based handling, and read the base revision of the fragment's header: to confirm no role refusal blocks the new exception's call, and that the header predates the diff.
