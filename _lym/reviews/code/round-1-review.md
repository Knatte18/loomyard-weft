---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    origin: lens:code-affected-set
    location: "crucible/orchestrator-prompt.md:119"
    summary: "Crucible smoke commands still select a tier with -tags, so under the run-time selector they run tier0 only and skip every smoke test green"
    fix: "In crucible/orchestrator-prompt.md lines 119 to 123, put LYM_TEST_TIER=2 in front of each go test command that passes -tags tier2 (LYM_TEST_TIER=3 where the line says -tags tier3 for an LLM-driving module), and export LYM_TEST_TIER=2 before the for-loop that runs the compiled smoke.test.exe copies, since a compiled binary reads the variable at run time. Keep or drop the -tags flag; it no longer selects anything."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: "internal/testkit/tmuxkit/tmuxallowed.go:9"
    summary: "tmuxAllowed moved from a build tag to the tier selection and no test pins it, so an inverted check would pass every gate"
    fix: "Add a table-driven test in package tmuxkit (an internal test file, or export tmuxAllowed through export_test.go) that sets LYM_TEST_TIER with t.Setenv (not parallel) and asserts tmuxAllowed() for unset and 1 and 0 (false), for 2 and 3 and 0-2 and 1-3 (true), and for junk (false). Each row opens with tiergate.Require(t, 0)."
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-tests
    location: "internal/testkit/tiergate/tiergate.go:62"
    summary: "Require failing the test on an unparsable LYM_TEST_TIER is untested, and it is the one path that keeps a typo from skipping every test silently"
    fix: "In internal/testkit/tiergate/tiergate_test.go add a row set that runs Require under a malformed value (tier1, 2-0) and asserts the test fails rather than skips, for example by running Require in a subprocess re-execution of the test binary with the variable set and checking a non-zero exit with the variable named in the output."
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-tests
    location: "cmd/testtiming/main.go:129"
    summary: "Nothing pins that testtiming exports the selection to its go children, so a dropped Setenv would run tier0 only and report green"
    fix: "Factor the three lines at main.go:129-135 that resolve the selection and call os.Setenv into one function that returns an error, and add a test in cmd/testtiming/main_test.go that calls it with tags tier1,tier2 and asserts os.Getenv of LYM_TEST_TIER is 1-2 afterwards (t.Setenv first so the process value is restored)."
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-tests
    location: "internal/impactset/guards.go:253"
    summary: "The Require-reading paths of RequiredTier, the tier-purity guard and the spawn scan are covered only by tag fixtures or the real-tree scan, so a regression there falls back to the tag path unnoticed"
    fix: "Add rows to the existing tables, no new test function. In internal/impactset/guards_test.go TestFindGuardTests add Require(t, tier) with a non-literal tier, a wrong package qualifier and a one-argument call, each expecting the tag fallback tier. In cmd/testtiming/spawnscan_test.go TestScanSpawns add one fixture whose test opens with tiergate.Require(t, 2) and expect out of process. In cmd/lym/tierpurity_test.go add fixture rows for the support-file skip and for a tier1 file referencing a tmuxkit spawn, run through the scan body split out as a function over source text."
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-affected-set
    location: "internal/battencli/testmain_integration_test.go:1"
    summary: "Removing battencli's tier0 TestMain drops the stub that refused a pair push, so a tier0 test reaching wire.go:980 or :1425 now pushes for real"
    fix: "In the TestMain of internal/battencli/testmain_integration_test.go keep the real push seam, but make the seam refuse when the selection holds only tier0: install the refusing stub (returning an error that says unit tests push no pair) when tiergate.Selected() reports Low == High == 0, and the counting real push otherwise."
  - id: F7
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: "internal/testkit/tmuxkit/tmuxkit.go:10"
    summary: "Comments and docs still describe build-tagged test binaries after the tier moved to run time"
    fix: "Reword to the run-time selector at these places. internal/testkit/tmuxkit/tmuxkit.go lines 10 and 18 (a tagged test, either tag). cmd/testtiming/main.go line 51 (a file with no tier tag). The headers of internal/battencli/testmain_integration_test.go, internal/hubgeom/testmain_integration_test.go, internal/loomcli/testmain_test.go, internal/websterengine/testmain_test.go and internal/hubforge/testmain_test.go. pattern/PATTERN-test-isolation.md line 14 (under every tag set), pattern/PATTERN-test-speed.md lines 151 and 167 to 168 (tested untagged, an untagged test is always judged), contracts/specs/loom-plan-spec.md line 340 (Untagged tests are fast), and the untagged wording in cmd/lym/tierpurity_test.go lines 27 and 59."
  - id: F8
    severity: LOW
    class: decision
    origin: handler
    location: "_lym/landing/summary.md:62"
    summary: "The change description gives the branch's per-tier pass counts but not main's, so the equality the no-silent-test decision asks for cannot be read from it"
    fix: "In the Verification section of _lym/landing/summary.md add main's passed counts for tier0, tier1 and tier2 from lym gate test records over the same targets, beside the branch counts, and name and explain each difference."
---

### [MEDIUM] Crucible smoke commands select a tier with -tags only

**Location:** `crucible/orchestrator-prompt.md:119`
**Issue:** The commands at lines 119 and 122 run `go test -tags tier2 … -run Smoke` and `go test -c -tags tier2`, and the loop at lines 123 to 124 runs the compiled binary. With `LYM_TEST_TIER` unset the selector picks tier0, so every smoke test skips and the run ends green. That is the silent skip the parent's no-silent-test decision rules out. The change description lists these commands as left alone, but the diff's own rewrite is what makes them skip.

### [MEDIUM] tmuxAllowed is not pinned by any test

**Location:** `internal/testkit/tmuxkit/tmuxallowed.go:9`
**Issue:** The old switch was an `init` in a `tier2 || tier3` file. The new one reads the selection. No `internal/testkit/tmuxkit/*_test.go` file sets `LYM_TEST_TIER`, so a reversed result would still pass: tier1 runs would stop flagging stray tmux servers, or tier2 runs would flag every registered one. I traced the function by hand for unset, 1, 2, 3, 0-2, 1-3 and an unparsable value, and it is correct today; the finding is the missing row.

### [LOW] Require's parse-failure path is untested

**Location:** `internal/testkit/tiergate/tiergate.go:62`
**Issue:** `TestSelected_ParsesTheVariable` covers the parser and `TestRequire_RunsOnlySelectedTiers` covers skip and run, but nothing calls `Require` on a value that fails to parse. That `t.Fatal` is what turns a mistyped variable into a red gate instead of an all-skipped green one.

### [LOW] testtiming's export of the selection is not pinned

**Location:** `cmd/testtiming/main.go:129`
**Issue:** `TestTierSelection` covers `tierSelection`. The `os.Setenv` that hands the result to every go child sits inline in `main`, and no test sees it. Dropping it would run tier0 only for every mode.

### [LOW] Require-reading paths covered by tag fixtures or the real-tree scan only

**Location:** `internal/impactset/guards.go:253`
**Issue:** `RequiredTier` rows cover the plain import, an alias, a tier2 file and not-first; the rejected shapes (non-literal tier, wrong qualifier, wrong arity) have none. The `spawnscan` fixtures carry `//go:build tierN` and no `Require`, so they exercise the tag fallback and not the reader every real file now uses. The purity guard's support-file skip and tier1 tmux-reference branches are reached only by the scan of the real tree, which is green with or without them. All of these are rows in existing tables, not new tests.

### [LOW] battencli loses the tier0 pair-push refusal

**Location:** `internal/battencli/testmain_integration_test.go:1`
**Issue:** The removed `!tier1` TestMain set `spawnPairPush` to return "unit tests push no pair". The kept one installs `fabricengine.PushPairAnchored`, and it now runs in every tier's binary. No tier0 test reaches `wire.go:980` or `wire.go:1425` today, but a future one would push for real instead of failing. The purity guard cannot see a call through a seam. The other 12 companions' packages lose nothing: their kept TestMain is the same or a superset (extra `HermeticGitEnv`, and `FABRIC_SKIP_PUSH=1` in hubreconcile, both harmless for tier0).

### [LOW] Stale tag wording in comments and docs

**Location:** `internal/testkit/tmuxkit/tmuxkit.go:10`
**Issue:** Lines 10 and 18 of the tmuxkit doc say "tagged test" and "either tag" while line 20 was already moved to the selection. The same wording remains in the files the fix lists, among them the `pattern/` background files this diff edited elsewhere. Line 14 of `PATTERN-test-isolation` states a rule over "every tag set" that no longer exists.

### [LOW] Main's pass counts are not recorded

**Location:** `_lym/landing/summary.md:62`
**Issue:** The parent decision asks that per-tier pass counts equal main's, or that each difference be named. The summary gives 22521, 3790 and 268 for the branch and nothing for main. I could not read main's records, so I checked the equality statically instead, and found no silent test. Across the 1632 modified test files, the tier named by every `Require` call equals the tier the file's old build tag gave it (negated terms ignored, a tag-less file read as tier0); the only apparent mismatches are TestMain files, which are not tests, and fixture source held in string literals. The module holds 5165 Test, Benchmark, Fuzz and Example functions at the base and 5155 at HEAD, and the ten fewer are the 13 removed TestMain companions plus the retired `TestIsTierTagged_RecognizesKnownTagsList`, less the new `tiergate` tests, its TestMain and `TestTierSelection`. `gate.yaml` is untouched, no per-tag fallback file exists, and no `.scratch` program is committed, as the amended record requires.

## Rejected

- Tier-purity guard skips support files (lens:code-correctness): the weakening is the known cost of one binary per package, and the change description and `pattern/PATTERN-test-speed.md` both state the bound.
- `Selection.String` untested (lens:code-tests): it feeds only the skip message and carries no contract.
- No Fuzz fixture row for `tierFailure` (lens:code-tests): Fuzz and Benchmark share one branch in `holdsTestFunction`, and the Benchmark row exercises it.
- No `test-tier-unstated` row with the new wording (lens:code-tests): the diff to `internal/loomshed/testtier.go` changes message constants only, and the matching code the existing rows test is unchanged.
- `Atoi` accepting `+1` and `01`, and `-tags tier-1` yielding `-1` (lens:code-correctness): neither selects a wrong tier; the second fails loudly at run time.
- `HermeticGitEnv` and `FABRIC_SKIP_PUSH` now present in tier0 runs (lens:code-correctness): benign widening of the environment, and the diff's purpose is one TestMain per package.
- `-tags` text in `internal/gatecli/doc.go` and the `planparser` verify-module checks (lens:code-affected-set): they describe the hub's current gate commands, which the change description says the orch changes after landing.
