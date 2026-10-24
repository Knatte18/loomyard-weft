---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md Decision 2 card-seed bullet and Decision 3 --card fallback bullet
    summary: The card seed omits a C# type glyph's own name and treats every non-.cs Target, inert docs included, as a whole-run fallback, so the card and batch gates run whole for the commonest Target shapes
    fix: In Decision 2's card-seed bullet, seed the glyph's Name as a type when its owner chain is empty (C# has no namespace-level members, so NORCE.Models#Circle names the type Circle), besides the owner chain of a member glyph or plan handle. In Decision 3's --card bullet, apply the diff form's path rules to a card Target's file, so a Target whose path is inert (Decision 5) is dropped from the seed, and only a non-.cs, non-inert file, a deleted file or a namespace-only self glyph runs whole. Add both cases to the card acceptance criterion.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md Constraints, PATTERN-config-layers bullet, and Decision 6 first paragraph
    summary: The template's build mapping keeps a hub's command, dir, env, timeout_min and about only if build is an open map or the template spells its sub-keys, and the record states neither, so the shape check or reconcile refuses or prunes a hub's build
    fix: In the PATTERN-config-layers Constraint, state that build joins gateslot.ConfigOpenMaps beside tiers and tier_parallel, so configengine carries its sub-keys whole through the layer stack and reconcile, and that gateslot.LoadConfig's refusals in Decision 6 validate its keys.
  - id: F3
    severity: MEDIUM
    class: design
    origin: handler
    location: _lyx/discussion/decision-record.md Decision 4
    summary: The new tier filter key applies at every scope, landing gates included, and VSTest exits 0 when a filter matches no test, yet Decision 4 states no bound on what a filter can let a tier skip
    fix: Add a Bound to Decision 4 stating that a tier's filter applies at whole scope too, so a filter matching no test passes the tier vacuously and writes a whole green entry; that it is bounded by sitting inside the executable, hub-file-only tiers key, whose command can already run anything; and that docs/dotnet-hub.md tells a hub that a landing tier's filter only excludes categories.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md Decision 2 lexer bullet and Bound
    summary: The lexer drops string literal content, so a type named only inside an interpolated-string hole is missed, which breaks the Bound's claim that only references never spelling the name are missed
    fix: In Decision 2's lexer bullet, state that the lexer keeps the code inside interpolated-string holes, regular and raw, as identifier tokens and drops only the literal text around them.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md Decision 2 result bullet
    summary: The filter term's spelling is left open while Decision 2 promises never under-matching, and a nested test class's FullyQualifiedName separates the outer class with a plus sign, so a dotted term matches none of its tests
    fix: In Decision 2's result bullet, state the term spelling as FullyQualifiedName~ followed by the namespace and the class chain, with a nested class joined to its outer class by + as VSTest reports it, and add a nested test class to the acceptance criterion on test class discovery.
  - id: F6
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md Decision 6 failed-build bullet and acceptance criterion on the build step
    summary: A failed build exits the call with the build's code only when no earlier run failed, since runTest keeps the first failure's code
    fix: In Decision 6's failed-build bullet and the matching acceptance criterion, replace the claim that the build is the call's first failing run with the plan-order rule, that the call exits with the first failed run's code, the build counting as a run before the opted-in tier it precedes.
  - id: F7
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md Decision 3 empty-diff paragraph and the inert_paths acceptance criterion
    summary: Decision 3 and the acceptance criterion say an empty dotnet diff runs no build, but a listed carry false tier with shared_build still runs whole there and Decision 6 runs the build before it
    fix: In Decision 3's empty-diff sentence and the inert_paths acceptance criterion, say an empty diff runs no build unless a listed carry false tier opts in with shared_build, which runs the build before it as Decision 6 states.
  - id: F8
    severity: LOW
    class: consistency
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md Decision 3 empty-test-scope paragraph and its acceptance criterion
    summary: When a closure selects no test class and one listed tier opts in with shared_build, the record leaves open whether a listed tier that is not opted in runs whole or runs nothing
    fix: In Decision 3's empty-test-scope paragraph and its acceptance criterion, state the per-tier rule, that an opted-in tier runs the build and no test command, and a tier not opted in runs whole with the reason on stderr, whether or not another listed tier opts in.
  - id: F9
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md Decision 4 {filter} bullets
    summary: Decision 4 picks the filter rendering by the call's scope, not the tier's share, so a whole-running tier inside a narrowed dotnet call, a carry false tier or a whole-fallback base group, has no stated rendering
    fix: In Decision 4, key the two {filter} bullets on the tier's own share, so a tier whose share is narrowed by the dotnet scoper renders the composed expression and every tier running whole renders its tier filter alone.
  - id: F10
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md Decision 2 test-project rule and Decision 3 fallback list
    summary: A selected test class with no ancestor .csproj, such as one under Models' .scratch directory, or in a directory holding two .csproj files, is silently not run, since Decision 3 falls back to whole only for a changed file
    fix: In Decision 2, add .scratch to the skipped directories, and in Decision 3 add a fallback to whole when a selected test class's file has no .csproj ancestor up to the tier's dir or its nearest ancestor directory holds more than one .csproj.
  - id: F11
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md Decision 5 first bullet and Decision 2 diff seed
    summary: The changed set is a git diff, which lists no untracked file, so under the none scoper a dirty tree with an untracked .cs file and only inert tracked changes now runs no tier where today it runs whole
    fix: In Decision 5's first bullet, state that the inert check counts untracked files, read from git status, as changed paths, so an untracked non-inert file keeps the scope whole; apply the same to Decision 2's diff seed.
  - id: F12
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md Open risks, first bullet
    summary: The risk claims a wide closure never costs more than today, but Decision 4 runs one dotnet test per selected project in sequence, where a solution run tests the projects in parallel
    fix: Rewrite the first Open risks bullet to say a closure selecting classes in every test project runs one dotnet test per project in sequence inside the tier, which can cost more than one solution-level run.
  - id: F13
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md tier1 acceptance criterion
    summary: A fixture solution whose projects form one reference chain may build in-process and never start an out-of-process MSBuild node, so the criterion would not exercise the node output-pipe hazard Decision 7 bounds
    fix: In the tier1 acceptance criterion, require the fixture to hold at least two projects MSBuild builds in parallel, built with -m, so an out-of-process node starts.
  - id: F14
    severity: LOW
    class: consistency
    origin: handler
    location: _lyx/discussion/support-log.md Question ledger Q2
    summary: Q2 names webster among the whole gates that catch a missed test, contradicting Decision 10 and Q12, which put webster at scope diff for a .NET hub
    fix: In Q2, name the whole gates as merge-in and finalize, with webster at scope diff per Q12.
  - id: F15
    severity: NIT
    class: consistency
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md Decision 10 second paragraph
    summary: Merge-In is the one whole-solution run per task only when no card, batch or webster gate fell back to whole under Decision 3
    fix: In Decision 10, qualify the sentence to say Merge-In is the one planned whole-solution run, besides the gates Decision 3 sends whole.
---

### [MEDIUM] Card seed misses type glyphs and inert Targets

**Location:** `_lyx/discussion/decision-record.md` Decision 2 card-seed bullet, Decision 3 `--card` fallback bullet

**Issue:** quarry v0.4.0's `glyph.Glyph` parses `NORCE.Models#Circle` with `Owner` nil and `Name` `Circle`, and `IsSelf` is false.
Decision 2 seeds only the file of a self glyph or the owner chain of a member glyph or handle, so a type Target seeds nothing and Decision 3 runs the card whole.
A card Target that is a doc file, such as `docs/x.md` landing in the same commit, also yields neither a `.cs` file nor an owner type, and Decision 5's inert rule covers only the diff forms.
Decision 10 relies on the card and batch gates narrowing, so both cases defeat the Goal at the most frequent gate.
Merged from walkthrough F1 and F2, bound-breaker F4, interactions F1 and the handler's own reading.
The plain-path half of walkthrough F1 is dropped: the plan spec canonicalizes a `.cs` path to its file self glyph.

### [MEDIUM] build is not an open map

**Location:** `_lyx/discussion/decision-record.md` Constraints `PATTERN-config-layers` bullet; Decision 6

**Issue:** `gateslot.ConfigOpenMaps` returns `tiers` and `tier_parallel` only, and `configengine` checks every layer's shape against the template outside an open map.
A template `build: {}` has no `command` key, so a hub's `build={command: ..., timeout_min: 5}` fails the shape check or is pruned at reconcile.
The Constraint's claim that the template seeding keeps a hub's values rests on a mechanism the record never names.

### [MEDIUM] Tier filter key has no stated bound

**Location:** `_lyx/discussion/decision-record.md` Decision 4

**Issue:** The rubric requires a Decision that adds a key able to let something through to state what it lets through and what bounds it.
`filter` renders at every scope, landing gates included, and Decision 4's own research says VSTest exits 0 when the composed filter matches no test.
A tier filter that excludes every test passes Merge-In vacuously and writes a whole green entry that later gates carry.
The go-json empty-tier check does not reach a vstest tier, and the plan-writer notes keep it go-json-only.

### [MEDIUM] Interpolation holes break the lexer's Bound

**Location:** `_lyx/discussion/decision-record.md` Decision 2 lexer bullet and Bound

**Issue:** Dropping string literal content drops the code in `$"{Formatter.Format(v)}"`, a reference that spells the type's name.
The Bound claims only references that never spell the name are missed, so the lexer rule contradicts it.

### [MEDIUM] Filter term spelling unstated

**Location:** `_lyx/discussion/decision-record.md` Decision 2 result bullet

**Issue:** Decision 2 promises terms that never under-match but leaves their spelling to Plan.
VSTest reports a nested class's FullyQualifiedName as `Ns.Outer+Inner.Test`, so a dotted term misses it.
This is a fact the design rests on, and the rubric's deferred-research rule requires it to be settled here.

### [LOW] Build failure exit code depends on plan order

**Location:** `_lyx/discussion/decision-record.md` Decision 6 failed-build bullet; acceptance criterion on the build step

**Issue:** `runTest` in `internal/gatecli/run.go` keeps the first failed run's code.
When tier0 does not opt in and fails, then tier1's build fails, the call exits with tier0's code, not the build's.
Merged from interactions F4 and bound-breaker F5.

### [LOW] Empty diff with a carry false opted-in tier

**Location:** `_lyx/discussion/decision-record.md` Decision 3 empty-diff paragraph; `inert_paths` acceptance criterion

**Issue:** Decision 8 runs a `carry: false` tier whole on every scope form but explicit targets, and Decision 6 runs the build before an opted-in tier whose command runs.
Decision 3 and the acceptance criterion say flatly that an empty diff runs no build.

### [LOW] Tier not opted in under an empty test scope

**Location:** `_lyx/discussion/decision-record.md` Decision 3 empty-test-scope paragraph; its acceptance criterion

**Issue:** The text reads as either "opted-in exists, so only the build runs" or "per tier".
A tier that is not opted in then either runs whole or runs nothing, since `scopeInvocations` makes no invocation for an empty module list.

### [LOW] filter rendering keyed on the call, not the tier

**Location:** `_lyx/discussion/decision-record.md` Decision 4 `{filter}` bullets

**Issue:** `scope.forTier` gives each tier its own share, so a narrowed `dotnet` call can hold whole-running tiers.
Decision 4 names only the call's scope, which leaves their `{filter}` unstated.

### [LOW] Selected class with no single ancestor project

**Location:** `_lyx/discussion/decision-record.md` Decision 2 test-project rule; Decision 3 fallback list

**Issue:** The board puts `.scratch/` under Models' project root, and Decision 2's skip list does not exclude it.
A selected class with no `.csproj` ancestor, or with two in its nearest directory, gets no project or an arbitrary one.
Decision 3's fallback covers only a changed file.
The whole gates bound the miss, so this is LOW.

### [LOW] Untracked files skip the inert check

**Location:** `_lyx/discussion/decision-record.md` Decision 5 first bullet; Decision 2 diff seed

**Issue:** `impactset` reads its changed set from `git diff --name-status`, which lists no untracked file.
Under `none` today a computed scope is always whole, so Decision 5 newly lets an agent's own computed call on a dirty tree skip an untracked `.cs` file.
Verify sites refuse a dirty tree, so this is LOW.

### [LOW] Open risk understates a wide closure's cost

**Location:** `_lyx/discussion/decision-record.md` Open risks, first bullet

**Issue:** Decision 4 runs one invocation per selected project, in sequence, inside one tier.
A solution-level `dotnet test` runs its projects in parallel, so "never more than today" does not hold when every project is selected.

### [LOW] tier1 fixture may never start a node

**Location:** `_lyx/discussion/decision-record.md` tier1 acceptance criterion

**Issue:** A one-chain fixture can build in the entry process, leaving the out-of-process node pipe hazard Decision 7 bounds untested.

### [LOW] Q2 contradicts Decision 10

**Location:** `_lyx/discussion/support-log.md` Question ledger Q2

**Issue:** Q2 says `webster`, `merge-in` and `finalize` are the whole gates.
Q12 and Decision 10 put `webster` at scope `diff` for a .NET hub.

### [NIT] One whole run per task is conditional

**Location:** `_lyx/discussion/decision-record.md` Decision 10

**Issue:** A diff touching a `.csproj`, `.resx` or test data sends card, batch and webster gates whole by Decision 3, so Merge-In is the one whole run only in the common case.

## Rejected

- Walkthrough F1's plain-path half: `loom-plan-spec.md` canonicalizes a plain `.cs` path to its file self glyph, which Decision 2 already seeds.
- Bound-breaker's "Constraint, `build: {}` in the template is shaped like `tiers: {}`: held": contradicted by `gateslot.ConfigOpenMaps`, which names only `tiers` and `tier_parallel` (kept as F2).
