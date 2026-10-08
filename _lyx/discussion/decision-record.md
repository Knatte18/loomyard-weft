## Goal

The plan gate's checks give no false findings and admit no unsatisfiable gate.
`caller-uncovered` resolves a reference by import path and receiver type instead of by name, so a re-signed method on one type no longer matches every same-named selector.
A gate command that tests packages inside a nested Go module runs inside that module, both in webster's Go-derived card gate and in the commands a plan author writes, and the plan gate refuses an author-written one that cannot pass.
The plan reviewer proposes no fix the plan gate blocks.

Sources: board issues #461, #451 and #460, and the `caller-scan-import-path` note folded into this task.

## Scope

In:

- `caller-uncovered` in `internal/planglyph`: type-resolved references, import-path resolution for package-level members, a collapsed name-only fallback, nested-module files scanned through that fallback.
- Webster's Go-derived per-card gate command in `internal/websterengine`: package directories inside a nested module are tested inside that module.
- A new plan check, `verify-nested-module`, in `internal/planparser`, over a card's `**Verify:**` value and the overview's `## verify:` section.
- One nested-module derivation in `internal/planparser`, shared by the card gate and the new check.
- The Plan-Review rubric (`contracts/stencils/loom/loom-rubric-plan-review.md`): a proposed fix must pass the gate's checks, and a nested module's packages count as covered by a command run inside it.
- The plan template stencil (`contracts/stencils/loom/loom-template-plan.md`): the `go -C` form for a nested module's packages.
- A `CLAUDE.md` line: a card whose new tests push a package past its `cmd/lyx/testdata/test-budget.yaml` row lists that file under `Edit:`.
- Same-commit docs: `contracts/specs/loom-plan-spec.md` (row 34 rewritten, a new row for `verify-nested-module`, the verify model's tier-1 bullet), `internal/planglyph/doc.go`, `internal/websterengine/doc.go`, `internal/planparser`'s doc where it lists checks.

Out:

- Any change to the quarry module (`github.com/Knatte18/quarry`); it has no reference API and stays untouched.
- References through an interface to a concrete method's interface counterpart, and the assignment sites a re-signed method breaks by no longer satisfying an interface.
- A caller in a file an earlier card creates; the file does not exist when the gate runs.
- Shell forms of a verify command beyond a literal `cd <dir>` segment and `go -C <dir>`.
- A lyx-level implicit target or check for `cmd/lyx/testdata/test-budget.yaml`.

## Decisions

### D1 — caller-uncovered resolves references with go/types through golang.org/x/tools/go/packages

Decision: `callerCoverageFindings` keeps its token pre-scan to find candidate files, then loads, through `golang.org/x/tools/go/packages`, the package directories holding a candidate plus each subject's declaring directory, with syntax, types and type info, `Tests: true` and build tags `integration,tmux,llm`, imports read from compiler export data.
The module gains `golang.org/x/tools` as a direct dependency.
In a file the load type-checked, an identifier is a reference to a subject when its `types.Info.Uses` object is that subject:
a package-level member by declaring package path, name and package-scope parent;
a method by declaring package path, receiver base type name (the generic origin for an instantiated receiver) and method name, so a promoted call through embedding counts and a call on another type, or through an interface, does not.
A method value (`x.start` without a call) is a reference.
A non-test file appears in both its package and the package's test variant, so references are deduplicated by file and line.
The declaration-span exclusion and the coverage and admissibility rules stay as they are.

Bound: a call through an interface whose method set holds the method is no longer reported, where today it is an informational name match.
A re-signed method that stops satisfying an interface breaks compilation at the sites that assign its type to that interface.
The card gate's `go build ./... && go test ./...` (D7) catches such a site in the root module's untagged code, and its `-tags integration` step only in the card's own directories.
A site in a tagged file elsewhere, or in a file built only for another GOOS, passes the card gate; the plan-level `## verify:` gate catches the tagged ones later, and no gate builds the other-GOOS one.
A nested module's assignment site is caught only by a gate command run inside that module.

Rationale: both observed false-positive classes (`batcher.Batch` read as a call of `costBatcher.Batch`, any `.start(` read as `peakAccumulator.start`) are receiver-type questions, which only a type checker answers.
`go/packages` is the Go team's loader for exactly this and handles test variants, cgo and export data; the repo already depends on other `golang.org/x` modules.
Loading only candidate directories keeps the load small.

### D2 — The load is a bounded, logged, offline spawn, and its failure degrades instead of erroring

Decision: the load runs with `Dir` set to the told worktree root, `GOPROXY=off` and `GOFLAGS=-mod=readonly` in its environment, under a 3-minute context timeout, and is logged as a spawn and its completion through `internal/logger`.
When the load fails, times out, or leaves a file or an identifier without type information (a file excluded by a build constraint such as another GOOS, a package with type errors), the occurrences in that file or at that identifier fall back to D3.
A load failure is never `ErrQuarryUnavailable`.

Bound: the fallback can let a method caller through as informational instead of blocking (D4), only where type information is missing, and the finding says the receiver was unresolved.
A package-level caller never weakens, because D3 resolves it exactly.

Rationale: before this task every method finding was informational and package-level matching had no type information at all, so the fallback is never worse than today.
Making the gate fail on a toolchain hiccup would block plans over a check whose job is advisory for methods.
`GOPROXY=off` keeps a plan gate from reaching the network.

### D3 — The fallback resolves package-level members by import path and drops package-qualified selectors for methods

Decision: in a file without type information, a package-level member is referenced:
inside its declaring package (same directory, same package clause) by its bare identifier not after `.`;
elsewhere through an import of the declaring package's import path, by the import's name (alias or default package name) followed by `.` and the identifier, or by the bare identifier when that import is a dot import.
The declaring package's import path is its enclosing module's path (from the nearest `go.mod` at or above its directory, up to the worktree root) plus its directory relative to that module.
With no enclosing `go.mod`, the package-clause rule of today applies.
A method is referenced by `.` and its identifier, except when the left side of the selector is a name the file imports.

Rationale: import-path resolution needs only the file's import block, so the alias and dot-import gaps of `caller-scan-import-path` close even without types.
Excluding import-qualified selectors removes the `batcher.Batch` class even on the fallback path.

### D4 — Severity follows resolution, and name-only matches collapse

Decision: a type-resolved or import-resolved reference is blocking, for a method as for a package-level member, one finding per subject and file as today.
Name-only method matches from D3 are informational and collapse into one finding per subject, naming every file with its lines and stating that the receiver could not be resolved.

Rationale: the informational severity existed only because a name scan cannot tell receivers apart; a type-resolved method caller is as certain as a package-level one, and leaving it uncovered fails webster's build.
Collapsing bounds the noise of the remaining name-only matches.

### D5 — Nested modules are scanned through the fallback

Decision: the reference walk keeps today's file set: it skips only the `testdata`, `vendor`, `.`- and `_`-prefixed directories, and scans a nested module's files.
The D1 load runs from the root module and leaves a directory inside a nested module unloaded, so every file there, a subject's declaring file included, has no type information and goes through D3.
A nested-module file that imports a root package by its import path references a root subject exactly as D3 states, and a subject declared in a nested module resolves through that module's `go.mod`.

Bound: inside a nested module a method caller is a name-only match and so informational (D4); a package-level caller stays blocking.

Rationale: a nested module can import the root module (a `replace ../` directive) and so hold a caller of a root subject, and a subject declared in a nested module has its callers there; skipping the directories would drop both, with no other gate building a nested module unless a card targets it.

### D6 — One nested-module derivation in planparser

Decision: `internal/planparser` exports one function answering, for a plan, a card number (or the whole plan) and the worktree root, the set of nested module directories: directories below the root holding a `go.mod` on disk or named by a `Create` target of a card numbered at or below the given one, minus those whose `go.mod` a `Delete` target of such a card removes.
A Rename pair moving a `go.mod` counts as a `Create` of its new side and a `Delete` of its old side.
The whole-plan form counts every card.
A directory belongs to the innermost nested module at or above it, or to the root module when none.

Bound: a `Move` card relocating a `go.mod` is not counted, because a `Move` states its destination in `Intent` prose rather than its target list.
For a card at or after such a `Move`, the set read at the plan gate puts the module at its old directory and the set read at a later dispatch puts it at its new one, so D7's card gate and D8's verdict for that card can differ between the plan gate and dispatch.
The plan model has the same `Move` blind spot for `path-missing`, accepted in the spec; the author closes it by moving a `go.mod` through a `Rename` pair instead.

Rationale: the card gate (D7) and the new check (D8) must agree on what a nested module is, including a `go.mod` a card creates; one function in the package that owns plan parsing serves both.
Counting created, deleted and renamed `go.mod` files makes the verdict for the changing card and every later card the same before and after that card runs; an earlier card's verdict can change after it, but it is done by then and its findings are dropped at dispatch.

### D7 — Webster's card gate tests a nested module's packages inside that module

Decision: `cardGateCommand` partitions the card's surviving package directories by D6's module for that card: root-module directories keep `go test -tags integration ./<dir> …`, and each nested module adds `go -C <module> test -tags integration ./<dir-relative-to-module> …`, joined into the chain with `&&` before the comment lint.
`go build ./... && go test ./...` at the root stays first and unchanged.

Rationale: the integration-tagged run builds and runs the untagged tests too, so one `go -C` command per nested module covers its packages; `go -C` needs no shell `cd` and keeps the command one chain.

### D8 — New blocking plan check `verify-nested-module`

Decision: `internal/planparser` reports `verify-nested-module` for a `go` command in a card's `**Verify:**` value or a line of the overview's `## verify:` section whose relative package argument (`.`, `./…`, `../…`) resolves into a nested module (D6: through that card for a `Verify:`, the whole plan for the overview) other than the module the command runs in.
The command runs in the root module, in the directory a preceding literal `cd <dir>` segment of the same `&&`/`;` chain set, or in `go -C <dir>`'s directory, which resolves against the directory a preceding `cd` set and against the root otherwise.
A flag's value (`-o ./bin`, `-run .`) is never a package argument.
The finding names the card or the overview, the command and the module, and its way forward is to run the command with `go -C <module>` and module-relative package paths.
It is blocking and runs wherever planparser's format checks run (both plan gates, `lyx loom validate-plan`, dispatch), under any `language:`, since it reads `go` commands, not glyphs.

Bound: the check only refuses; it misses a command built from a variable, inside `bash -c`, after a subshell, or with an import-path package argument, and webster's gates still catch those at run time.
After a `Move` of a `go.mod` (D6's bound) its verdict on a later card can flip between the plan gate and dispatch.

Rationale: issue #451's plan passed the gate with gates unsatisfiable by construction; with the card gate fixed (D7), the author-written commands are the remaining place such a command can appear.
Its verdict reads plan text and file existence only, so it is stable across the run except for D6's `Move` bound, and it belongs in planparser beside `path-missing`.

### D9 — The plan reviewer checks a proposed fix against the gate's checks

Decision: the Plan-Review rubric gains a rule that binds the proposed fix, never the finding: a finding whose fix adds, drops, moves or pairs targets, or changes a verify command, proposes only a fix every check in `{{.specs_dir}}/loom/loom-plan-spec.md`'s validation-checks section admits.
When the reviewer sees no admissible fix, it still reports the defect and names the check that constrains the fix.
The `**Fix:**` line is a proposal: the fixer may apply any other fix the checks admit, and when none exists it defers the finding in its fixer-report with the constraining check as the reason, under burler's existing fix rule.
The rubric points at that section and copies no check list.

Bound: the rule removes only a blocked fix shape from what a reviewer proposes; no defect becomes unreportable, so a defect the checks miss still reaches the fixer.
A deferred finding with no admissible fix keeps the segment from converging on it, and the judge's escalation takes it to the parent session and the operator.
Its verify-coverage rule counts a nested module's packages as covered by a command run inside that module (`go -C <module>`).

Rationale: the reviewer already had a pointer to the check list, but only for what not to flag; the wasted round came from a proposed fix, so the rule must bind the fix.
Pointing rather than copying follows `PATTERN-producer-pointer-rule`.
A blocked proposed fix does not make the defect's premise false, so the fixer cannot dispute it; deferring it names the decision the operator must make.

### D10 — The plan template states the nested-module form

Decision: `loom-template-plan.md`'s verify guidance tells the writer that a nested module's packages are tested with `go -C <module>` and module-relative paths, pointing at the spec's `verify-nested-module` row rather than restating it.

Rationale: the writer/reviewer symmetry note in the rubric requires the writer to be told what the reviewer and gate hold it to.

### D11 — The test-budget row is a repo instruction in CLAUDE.md

Decision: `CLAUDE.md` gains one line: a plan card whose new top-level tests push a package past its row in `cmd/lyx/testdata/test-budget.yaml` lists that file under `Edit:`.

Rationale: the budget file is this repository's own guard, so lyx's generic stencils and checks must not name it; every agent in the repo, the plan writer included, loads `CLAUDE.md`.
An untargeted edit to it is already only an informational deviation, so a written rule is proportionate.

## Constraints

- `PATTERN-told-geometry`, `PATTERN-cwd-resolution`: the package load takes the told worktree root as `Dir`; nothing in planglyph or planparser derives a path or calls `os.Getwd`.
- `PATTERN-spawn-observability`: the `go list` spawn behind `go/packages` is logged before the load and at its end through `internal/logger`.
- `PATTERN-test-speed`: a test that runs the real load spawns `go list`, so it sits in the `integration` tier; untagged tests do not spawn and reach the fallback or the scan through an injected loader or pure inputs.
- `PATTERN-test-isolation`: planglyph's integration tests keep running through its existing `tmuxkit.Main`.
- `PATTERN-sole-parsers`: the verify-command scan of D8 lives in `internal/planparser`, which alone reads plan files.
- `PATTERN-gate-self-check-parity`: D8 joins planparser's existing format checks, so `lyx loom validate-plan` and the plan gates keep calling the same function.
- `PATTERN-producer-pointer-rule`: the rubric and template point at the spec's validation-checks section and rows and copy none.
- `PATTERN-refusal-way-forward` (by analogy for plan findings): every new or changed finding names its way forward.
- `PATTERN-test-economy`: new coverage extends the existing glyphchain fixture and its tests, and websterengine's existing card-gate tests, before adding test files.
- `PATTERN-documentation-lifecycle` and CLAUDE.md's "Docs land in the same commit": the spec rows and both `doc.go` files change with the code.
- Planglyph's chokepoint pins stay intact: no new `Resolve` or `quarry.Name` call site.
- Build stays cgo-enabled; `golang.org/x/tools` adds no cgo of its own.

## Auto-mode assumptions

- No operator answered questions; every pick below was made on judgment and is logged as an auto-pick in the support log's question ledger.
- Adding `golang.org/x/tools` as a direct dependency is acceptable (D1).
- Making a type-resolved method caller blocking is wanted, though it can surface blocking findings on plans that passed before (D4).
- 3 minutes is enough for a load on a warm build cache and a tolerable wait on a cold one (D2).
- The test-budget concern is met by a `CLAUDE.md` line rather than a lyx mechanism (D11).
- The reviewer half of issue #460 is met by the rubric rule; no mechanical validation of a reviewer's proposed fix is built, because a proposed fix is prose.

## Open risks

- A cold build cache makes the first load compile the dependency graph, tree-sitter's cgo grammars included; a timeout then degrades method findings to informational for that gate run.
- A package that fails to type-check leaves identifiers without objects; D2's per-identifier fallback covers them, but a badly broken package yields mostly name-only matches.
- A blocking method finding is new; plan writers may need a round to learn to list method callers, which the glyph-practice audit (`glyph-practice-audit`) will see in Plan rounds.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 2: converged; each finding below was addressed by the round's fixer, and no fresh reviewer has seen the fix.
  Review: _lyx/reviews/discussion/round-2-review.md; fixer report: _lyx/reviews/discussion/round-2-fixer-report.md.
  - d6-move-of-gomod-uncounted (class design, severity MEDIUM, open in rounds 2)
  - d9-dispute-route-forbidden-by-fix-rule (class design, severity MEDIUM, open in rounds 2)
<!-- /lyx:carry-over Discussion-Review -->

## Acceptance criteria

- A re-signed package-level member referenced through an aliased import, and through a dot import, is reported blocking.
- A re-signed method `T.M` is not reported for a call `u.M()` on another type with a method `M`, nor for a package-qualified identifier `pkg.M` naming a type or function; this reproduces both classes of issue #461.
- A call of `T.M` promoted through an embedding struct, and a method value `t.M`, are reported blocking.
- A call through an interface whose method set holds `M` is not reported for `T.M`.
- A reference in a file the load does not type-check (a file constrained to another GOOS) falls back: a package-level member resolves by import path and stays blocking; a method name match is one collapsed informational finding per subject that names the files and lines and says the receiver was unresolved; a selector on an imported package name is not a match.
- When the load fails outright, the gate still answers with the fallback findings and returns no infrastructure error.
- In a nested module outside `testdata`, a file importing a root package by its import path is scanned through the fallback: a package-level caller of a root subject there is reported blocking, and a method name match there is informational.
- The card gate for a card targeting a directory inside a nested module, existing, created by a `go.mod` `Create` target on that card or an earlier one, or moved there by such a card's Rename pair, runs `go -C <module> test -tags integration` with module-relative paths, and leaves root-module directories as before.
- `verify-nested-module` fires on a `Verify:` value and on a `## verify:` line that tests a nested module's package from the root, and stays silent for the `go -C <module>` form, for a preceding `cd <module>` segment, for a flag value such as `-o ./bin`, and for a card's `Verify:` when the creating card comes after it.
- `lyx loom validate-plan` reports `verify-nested-module` exactly as the plan gates do.
- The Plan-Review rubric binds a proposed fix to the spec's validation-checks section, keeps the finding legitimate when no admissible fix exists and has the reviewer name the constraining check, and counts `go -C` as covering a nested module's packages; the plan template states the `go -C` form; existing stencil-claim tests pass with the new wording.
- The spec, `internal/planglyph/doc.go` and `internal/websterengine/doc.go` describe the new behavior, and the old bound about aliased and dot imports is gone from row 34.

## Notes for the plan writer

- The glyphchain fixture (`internal/planglyph/testdata/glyphchain`) is already a nested module with callers; extending it with an aliased importer, a dot importer, a same-named method on another type, an interface call and an embedding covers D1–D4 in one place.
- `cardGateCommand` in `internal/websterengine/cardgate.go` builds the chain today from `planparser.CardTargetDirs`.
- Compare an instantiated generic receiver through `types.Func.Origin` and the named type's `Origin`.
- `issue #451`'s second symptom is D11; its first is D7 and D8.
