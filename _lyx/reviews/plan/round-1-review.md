---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:guard-reach
    location: _lyx/plan/10-inert-paths.md:26
    summary: card 10 gives an inert-only base group the empty share but never says how that share passes computeScope's closing switch, which turns an all-inert diff under go into a whole scope, or into a guard step with an empty guard map
    fix: 'In card 10''s Narrowed gates bullet, add that under go an inert-only group counts as narrowed and leaves the guard tests to the module scan, so an all-inert call returns a scope with the per-tier shares, guardStep set, guardDir tier0''s dir and guards nil, as the anyWhole branch of computeScope does today, never the !narrowed whole return and never guardsByTier over the nil guards of skipped derivations; under none, the early whole return becomes a per-group share in which only an inert-only group is empty, the others whole, with guardStep false.'
  - id: F2
    severity: MEDIUM
    class: consistency
    origin: lens:guard-reach
    location: _lyx/plan/08-dotnet-scoper.md:55
    summary: the C# seed tree of card 8's TestGateTest_Scopes rows holds no .csproj above cs/Lib/Circle.cs and no production type that no test class names, so under card 5's project rule every narrowed dotnet row falls back to whole, and card 11's no-test-class rows cannot reach noTests
    fix: In card 8's TestGateTest_Scopes paragraph, name the full seed tree as cs/Lib/Lib.csproj, cs/Lib/Circle.cs, cs/Lib/Unused.cs declaring a type Unused that no test class names, cs/Lib.Tests/Lib.Tests.csproj, cs/Lib.Tests/CircleTests.cs naming Circle and cs/Lib.Tests/OtherTests.cs; in card 11's TestGateTest_Scopes paragraph, say its edit to a type no test class names is the edit to cs/Lib/Unused.cs from card 8's tree.
  - id: F3
    severity: LOW
    class: consistency
    origin: lens:way-forward
    location: _lyx/plan/08-dotnet-scoper.md:40
    summary: card 8 promises a dotnet --guards refusal that names only dropping --guards, but the refusal is built through configRefusal, which always appends the lyx config gate way forward, and no config change helps a C# hub
    fix: 'In card 8''s --guards bullet, say the dotnet form is not built through configRefusal and reads, exactly: gate test: --guards needs the go scoper to find the guard tests, but gate.yaml''s scoper is "dotnet"; way forward: drop --guards, then re-run the same command. The none form keeps its configRefusal text, and the TestPlanTierRuns row asserts the dotnet text exactly.'
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lyx/plan/06-derive-entry-points.md:53
    summary: card 6's Intent makes a failed git read a Whole fallback, as Decision 3 lists, while its Tests paragraph calls a failed git diff or git show a propagated git error, so the card disagrees with itself on the branch's behaviour
    fix: Replace card 6's sentence at line 53 with one saying that a failed git diff, git ls-files or git show, apart from a path absent at the base, which reads as declaring no type, returns Whole naming the git error, as Decision 3 requires, and that no row forces one, since a local git read failing over a valid repository is a transient I/O failure no fixture can produce.
  - id: F5
    severity: LOW
    class: design
    origin: lens:way-forward
    location: _lyx/plan/11-shared-build-step.md:46
    summary: card 11 plans no refusal for a build step that cannot start, such as a build dir that does not exist, so it fails as a bare exit 1 with an empty log, while a tier in the same state is refused naming its way forward
    fix: In card 11's runBuild bullet, add that a build whose shell cannot start, its process nil while ctx is not done, is refused as runInvocations refuses a tier, naming the build's dir and the way forward make build.dir an existing directory and put the platform shell on PATH, then re-run the same command; add a TestGateTest_Scenario row with build dir set to a missing directory asserting that refusal.
  - id: F6
    severity: LOW
    class: consistency
    origin: lens:way-forward
    location: _lyx/plan/02-build-run-lines.md:46
    summary: card 2 swaps only the tier rewrite in verifyGate.verdict's timed-out reason for a build timeout and keeps its opening step, fix the hanging test on the task branch, although a build timeout ran no test
    fix: In card 2's verifyGate.verdict bullet, say that for a build timeout the reason's first step reads fix the hanging build, such as a stalled restore or compile, on the task branch, in place of fix the hanging test; extend the TestVerifyGate_CheckMapsVerifyResult build row to assert that text.
  - id: F7
    severity: LOW
    class: scope
    origin: handler
    location: _lyx/plan/11-shared-build-step.md:51
    summary: Decision 6 has each blocked opted-in tier write a run record naming the build failure and its log, but card 11's reportBlocked record carries only outcome failed and the build's exit code
    fix: In card 2, add to RunRecord a field BlockedBy string with tag json:"blocked_by,omitempty", the log path of the failed build that kept the tier from running; in card 11's reportBlocked bullet, set it on each blocked record, and have the TestGateTest_Scenario build-exits-3 row assert it names the build's log.
  - id: F8
    severity: LOW
    class: consistency
    origin: handler
    location: _lyx/plan/08-dotnet-scoper.md:38
    summary: card 8 wires the dotnet narrowing into lyx gate test, which lets a gate skip test classes, but its Intent never states what the narrowing can skip or what bounds it
    fix: Add a sentence to card 8's Intent saying the narrowing skips every test class the name closure does not select, so it misses only references that never spell a changed type's name, as Decision 2's Bound lists, and that those misses are bounded by the whole runs at merge-in and finalize and by the whole fallbacks of Decision 3.
  - id: F9
    severity: LOW
    class: consistency
    origin: handler
    location: _lyx/plan/08-dotnet-scoper.md:41
    summary: card 8 sets partial on every invocation of a narrowed dotnet share, which includes the one whole run of a command without {targets}, so a pass that ran whole writes no green entry although the green-record Constraint withholds an entry only from filtered or project-narrowed runs
    fix: In card 8's scopeInvocations bullet, say partial is set only on the per-project invocations, and the single whole invocation of a command without {targets} leaves it unset; make the TestPlanTierRuns row for a command lacking {targets} assert partial unset.
  - id: F10
    severity: LOW
    class: consistency
    origin: lens:affected-set
    location: _lyx/plan/11-shared-build-step.md:21
    summary: lyx gate test's help text says each tier prints a summary line and writes its record, but no card adds the build step's line and record to it, nor the --guards refusal under dotnet
    fix: Add the glyph internal/gatecli#newCommand to card 11's Edit list and a bullet saying its Long help gains one sentence naming the build step, its own summary line and its run record, and that --guards is refused under the none and dotnet scopers.
  - id: F11
    severity: NIT
    class: consistency
    origin: lens:affected-set
    location: _lyx/plan/01-gate-config-keys.md:27
    summary: card 1 adds dotnet to the scoper but does not name the Config.Scoper field comment, which still reads go or none
    fix: Add to card 1's scoper bullet that Config.Scoper's comment names go, none or dotnet.
  - id: F12
    severity: NIT
    class: consistency
    origin: lens:way-forward
    location: _lyx/plan/07-filter-placeholder.md:25
    summary: a tier filter holding a percent sign, which the Windows quoting refuses, gets the quoteAll way forward pass a value without a percent sign, which addresses the caller's words while the value sits in the hub's gate.yaml
    fix: In card 7, say filterArgument wraps a quoting error of the tier's filter in a refusal naming the tier and the way forward fix the tier's filter with lyx config gate from the prime, closing with the role clause.
  - id: F13
    severity: NIT
    class: consistency
    origin: handler
    location: _lyx/plan/07-filter-placeholder.md:23
    summary: card 7 bounds the joined terms by greenCommandBudget where the added decision bounds the rendered filter, and its Info log names the tier although filterArgument is told no tier number
    fix: In card 7's budget bullet, compare the length of the composed expression, tier filter included, against greenCommandBudget, and either pass the tier number to filterArgument or drop the tier from the Info log, which then names the dropped term count only.
---

### [MEDIUM] Inert-only shares do not survive computeScope's closing switch

**Location:** `_lyx/plan/10-inert-paths.md:26`

**Issue:** Card 10 gives an inert-only base group the empty share and skips `impactset.DeriveTargets` for it.
It never says how that share passes the switch that ends `computeScope` (`internal/gatecli/scope.go`).
When no group set `narrowed`, the switch returns `scope{whole: true}`, so an all-inert diff under `go` would run every listed tier whole.
If the inert group instead counts as narrowed with no whole tier, the last branch builds `guardsByTier(nil)`, a non-nil empty map, so `runTest` skips the module scan and the guard step runs nothing.
Either path defeats the card's own `docs/note.md` row, which expects no tier command and a guard step.
Under `none`, the early whole return has to become per-group shares, which the card also leaves unstated.

### [MEDIUM] The C# seed tree cannot reach the narrowed scopes its rows assert

**Location:** `_lyx/plan/08-dotnet-scoper.md:55`

**Issue:** The tree lists `cs/Lib/Circle.cs` beside only `cs/Lib.Tests/Lib.Tests.csproj`.
Card 5's `narrow` sends a seed file with no `.csproj` in an ancestor directory up to the tier's `dir` to `Whole`, as Decision 3 requires.
So the computed, `--base` and `--card` rows can never run `Lib.Tests.csproj` with the `CircleTests` term.
Card 11's row editing "a type no test class names" (`_lyx/plan/11-shared-build-step.md:68`) builds on the same tree.
That tree holds no such type and falls back to whole, so it never reaches `noTests` or `buildOnly`.

### [LOW] The dotnet --guards refusal keeps a config way forward

**Location:** `_lyx/plan/08-dotnet-scoper.md:40`

**Issue:** `configRefusal` (`internal/gatecli/tiers.go`) appends `; way forward: fix it with "lyx config gate" from the prime` to every message it builds.
Rewording only the format string still sends a C# hub's agent to the config, which the card itself rejects as a way forward.

### [LOW] Card 6 describes a failed git read two ways

**Location:** `_lyx/plan/06-derive-entry-points.md:53`

**Issue:** Line 36 lists a failed git read among the `Whole` fallbacks, and says a cancelled `ctx` is the one error.
Line 53 calls a failed `git diff` or `git show` a propagated error.
An implementer can satisfy only one of the two, and `git show` of a path absent at the base must stay a missing version, not a failure.

### [LOW] A build that cannot start has no way forward

**Location:** `_lyx/plan/11-shared-build-step.md:46`

**Issue:** `runInvocations` refuses a tier whose shell could not start, naming the dir and the shell (`internal/gatecli/tierrun.go`).
`runBuild` has no such branch, and card 1 does not validate `build.dir`.
A missing build directory therefore reaches `exitCodeOf` as a plain exit 1 with an empty build log.

### [LOW] The build-timeout landing reason still points at a hanging test

**Location:** `_lyx/plan/02-build-run-lines.md:46`

**Issue:** `verifyGate.verdict`'s timed-out reason opens with "fix the hanging test on the task branch" (`internal/landingshed/verifygate.go`).
Card 2 replaces only the trailing rewrite for a build, so the step the landing fixer reads first names a test that never ran.

### [LOW] Blocked tiers' run records do not name the build failure

**Location:** `_lyx/plan/11-shared-build-step.md:51`

**Issue:** Decision 6 says every opted-in tier left in the call "fails without running its command, each with a stderr line and run record naming the build failure and its log".
Card 11 names the log only on the stderr line.
`RunRecord` (`internal/gateslot/runlog.go`) has no field that names the build or its log, so a blocked record reads as a failed tier run with no cause.

### [LOW] Card 8 states no bound for the narrowing it wires

**Location:** `_lyx/plan/08-dotnet-scoper.md:38`

**Issue:** Card 8 is the card through which a `dotnet` gate first skips test classes.
The bound on that skip appears only in card 5's package doc, and card 8's Intent does not state what the routing edge can skip or what catches its misses.

### [LOW] partial withholds the green entry of a whole run

**Location:** `_lyx/plan/08-dotnet-scoper.md:41`

**Issue:** Card 8 sets `invocation.partial` on every invocation of a narrowed `dotnet` share.
Under such a share, a command without `{targets}` runs once, whole, through `scopeInvocations`' whole branch.
`recordPass` would then skip a pass that covered every test, which the Constraint does not ask for.
This loses a later carry, though it never runs anything unsafely.

### [LOW] The gate test help text misses the build step

**Location:** `_lyx/plan/11-shared-build-step.md:21`

**Issue:** `newCommand`'s Long help (`internal/gatecli/cli.go`) says each tier prints a summary line and writes its record and log.
Card 11 adds a build line and record, and card 8 adds the `dotnet` `--guards` refusal, but only `reportCmd`'s help is edited.
`validate-plan --overlay` with the proposed Edit target added reports no blocking finding.

### [NIT] Config.Scoper's comment still says go or none

**Location:** `_lyx/plan/01-gate-config-keys.md:27`

**Issue:** `internal/gateslot/config.go`'s `Scoper` field comment reads "go or none", and card 1 names the parallel `Tier.Command` comment but not this one.

### [NIT] A percent sign in a tier filter gets the caller-words way forward

**Location:** `_lyx/plan/07-filter-placeholder.md:25`

**Issue:** `quoteAll`'s refusal tells the caller to pass a value without a percent sign, but a tier `filter` comes from `gate.yaml`, which only the prime changes.

### [NIT] The filter budget measures the terms, and the log names an unknown tier

**Location:** `_lyx/plan/07-filter-placeholder.md:23`

**Issue:** The added decision bounds the rendered filter, but the card measures only the terms joined by `|`.
`filterArgument(tier gateslot.Tier, terms []string)` receives no tier number, so it cannot name the tier in its Info log.

## Rejected

- guard-reach F3, the `NoCarry` subtract guard as unreachable: Decision 8 says the green record never carries a `carry: false` tier, and the guard expresses that rule directly; the rubric counts a guard reached by a named test, here a `TestGreenRecord` subtract row, as reachable.
- guard-reach F5, `DeriveDiff`'s empty-base fallback as unreachable from `gatecli`: Decision 3 lists the fallback, and `TestDeriveDiff`'s empty-base row reaches it through the package's exported entry point.

## Focus questions with no finding

- The focus directive's card-order question: no card uses what a later card creates, and card 10 reads nothing of card 11, so there is no finding.
- The focus directive's granularity and test-economy question on cards 4 to 6: each is one reviewable unit with its own pure-core or entry-point test, and no planned test duplicates an existing one, so there is no finding.
- The focus directive's Verify-line question: no card carries a `**Verify:**` line, so there is no finding.
