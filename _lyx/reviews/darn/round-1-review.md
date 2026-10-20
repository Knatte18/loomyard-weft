---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    origin: lens:code-affected-set
    location: internal/impactset/derive.go:200
    summary: The impacted-set command the round gate derives still runs the untagged go test and then the integration go test over the same set, so the unit tier of the impacted packages runs twice.
    fix: In buildCommand in internal/impactset/derive.go, drop the untagged "go test "+args step so the set runs only "go test -tags integration "+args after the vet steps, update the buildCommand comment and the command description at internal/impactset/doc.go:28 to say the integration run covers every untagged test, and update the want commands in internal/impactset/derive_test.go (lines 35-36, the gateCommand-style helper) and internal/impactset/derive_integration_test.go that spell the untagged step.
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/gatecli/prebuild.go:127
    summary: A prune running between another run's existence check and its refresh can still delete the entry after the refresh, because the not-exist retry covers only a removal before os.Chtimes; the refreshing run then exports a path that no longer exists.
    fix: In storePrebuilt in internal/gatecli/prebuild.go, after a successful os.Chtimes refresh, os.Stat the stored path once more and treat a not-exist result as not refreshed so copyIntoPlace republishes it; the window is a prune that read the entry's old modification time before the refresh and removes it after.
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/gatecli/prebuild_test.go:1
    summary: The branch that republishes an entry a concurrent prune removed between the existence check and the refresh, which commit d8e0b823 added, has no test, so returning the error there would pass every test.
    fix: Give storePrebuilt an injectable step between its os.Stat and its os.Chtimes, or split the refresh into a small function a test can call with a missing path, and add a TestStorePrebuilt row where the entry is removed in that window and the call returns the content-named path with the binary present and no staging file left.
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/loomshed/verifygate_integration_test.go:249
    summary: No test pins the assembled gate command when a record yields failing tests and no package pass, as for a plan_verify record or a publish_verify record with no impacted packages, so the failedTests-plus-command assembly with an empty package pass is covered only through the unit table of the two pieces.
    fix: Add a scenario to TestVerifyGate_RoundScenario in internal/loomshed/verifygate_integration_test.go that writes a plan_verify Publish failure record naming one failing test and asserts verifytree.LatestPass finds a pass of "go test -tags integration -run '^TestB$' example.com/m/b && " followed by the derived command, with nothing after it.
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:code-pattern-fit
    location: pattern/PATTERN-test-speed.md:37
    summary: The diff adds two cross-cutting rules, a content-stable exported prebuilt path and no per-invocation value in go test's environment, that live only in package docs, while CLAUDE.md says a new cross-cutting invariant goes into PATTERN in the same commit.
    fix: In the "Every gate run takes a slot" section of pattern/PATTERN-test-speed.md, after the sentence on the prebuilt lyx, add one sentence stating that a tagged hub run exports a path named by the binary's content and that lyx gate test gives go test no per-invocation value such as the trace id, since Go keys a cached test result on every variable a test reads.
  - id: F6
    severity: NIT
    class: consistency
    origin: lens:code-pattern-fit
    location: CLAUDE.md:37
    summary: The added clause joins independent clauses with semicolons on one line instead of breaking the line at each clause boundary, against the semantic-line-break rule for markdown.
    fix: In the push-rule bullet at CLAUDE.md:37, break the line before "the `integration` tag compiles every untagged test too" and before "code under `tools/` follows PATTERN", keeping the bullet's indentation on the continuation lines.
---

### [MEDIUM] The impacted-set round command still runs the unit tier twice

**Location:** internal/impactset/derive.go:200
**Issue:** The board entry's first part asks that verify, the darn verify command and the loom verify gates alike, stop running the untagged tier before the `integration` tier, because the tag already compiles and runs every untagged test.
The diff makes that change in `CLAUDE.md`, `pattern/PATTERN-test-speed.md` and the `internal/languageconfig` doc, but `buildCommand` in `internal/impactset` still emits `go test <set>` and then `go test -tags integration <set>`.
That command is the Webster-Burler-Fix round gate's derived command, run through `NewVerifyGate`, and the `PATTERN-test-speed.md` row for that gate still names it as "the impacted-set command".
The impacted packages' unit tests therefore still run twice and link two test binaries per package, the cost this task exists to remove.

### [LOW] A prune can remove an entry after another run has refreshed it

**Location:** internal/gatecli/prebuild.go:127
**Issue:** `pruneStore` reads an entry's modification time and removes the entry in two steps.
A run reusing an entry idle for more than the retention can `Stat` and `os.Chtimes` it between those two steps, so the prune removes the entry the run just refreshed.
The not-exist retry handles only a removal before the `os.Chtimes`.
The run then exports a missing path, and its `lyxbin` tests fail with the stale-prebuilt error, a red verify on an unchanged tree.
The window needs an entry idle for the retention and a prune running at that moment, so the cost is rare and the entry's idle time bounds it.

### [LOW] The republish branch of a concurrent prune is untested

**Location:** internal/gatecli/prebuild_test.go:1
**Issue:** `TestStorePrebuilt` reaches the refresh-succeeds path and, through `copyIntoPlace`, the lost-rename-race path.
Nothing reaches the branch at `internal/gatecli/prebuild.go:127-134` where `os.Chtimes` reports not-exist after the `os.Stat` succeeded, nor the branch that returns any other `os.Chtimes` error.
The summary's hand-check list names the first branch as the point of commit `d8e0b823`, so a regression that returned the error there would pass every test.

### [LOW] The gate's assembled command is untested with failing tests and no package pass

**Location:** internal/loomshed/verifygate_integration_test.go:249
**Issue:** Both integration scenarios use a `publish_verify` record with a non-empty impacted-package list, so each pins the form with both a failing-test prefix and a package-pass suffix.
`TestPublishFailure` pins `publishFailureSteps` returning a failing-test string with an empty package pass, but not the assembly in `NewVerifyGate` that prepends it and appends nothing.

### [LOW] Two new cross-cutting rules are written only in package docs

**Location:** pattern/PATTERN-test-speed.md:37
**Issue:** Breaking either rule, a path that changes per run or a per-invocation variable left in go test's environment, silently turns every affected package's cached result into a miss, and nothing but the package docs states them.
`CLAUDE.md` puts a new cross-cutting invariant into PATTERN in the same commit as the code.

### [NIT] The push-rule bullet packs three clauses into one line

**Location:** CLAUDE.md:37
**Issue:** The added clause follows the existing semicolon-joined clauses on the same line, so a later edit to one clause rewrites the whole bullet.

## Rejected

- The `internal/gateslot/doc.go` sentence says the `PrebuiltDir` join spells the ephemeral directory name through `internal/lyxdirs` (lens:code-pattern-fit, NIT): the sentence's claim is that the package derives no path beyond its joins, which holds, and the wording slip changes no reader's action.

- Prepending the failing tests assumes the told command is a single `&&` chain (lens:code-correctness, LOW): `contracts/specs/loom-plan-spec.md:289` has the parser chain every verify line with ` && `, the derived command is built the same way, and the previous append had the same shape, so there is no regression.
- `verifytree.Verify` strips the prebuilt variable but not the trace id, so the raw `go test` steps always miss the cache (lens:code-correctness, NIT): the diff does not touch it, and the board entry scopes the trace-id strip to `lyx gate test`.
- The `contracts/stencils/loom/loom-rubric-plan-review.md:103` sentence on the untagged and integration gates (lens:code-affected-set, LOW): it describes the separate card and batch gates, which this diff does not change, and it contradicts nothing.
- `TestStorePrebuilt` repeats its reuse-and-refresh block in every row, and `CopyIntoPlaceFailsOnAMissingSource` tests an unexported helper (lens:code-tests, NIT): the repeated block is a cheap shared assertion of one behavior, and the direct helper test reaches a cleanup the surface cannot, as that lens itself notes.
- The literal `LYX_TRACE_ID` in `internal/testkit/tmuxkit/tmuxkit.go:388`: the diff reverted that import in `ff8d5a0f` because it closes an import cycle, and the remaining literals are in tests and comments.
- Concurrent rename or prune replacing an entry another gate still uses: a rename replaces an entry of identical content and a running holder keeps its old inode, and a fresh entry is never pruned, so I found no way to lose a live binary beyond F2.
- Store path derivation: `Pool.PrebuiltDir` joins the pool's told directory with a name, so there is no new `.lyx` literal, and the prebuild stays inside the held slot with its build output under the `lyxtemp` run directory.
- Summary and benchmark claims: the `!integration` files are TestMain shims with tagged counterparts, the benchmark rows match the figures in the summary, and the `internal/shedbuild` fixture change matches its description.
