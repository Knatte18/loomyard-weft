---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/gatecli/only.go#markedScope
    summary: With the unlisted-tier refusal gone, a test marked for a key in a file of a manual tier (one that spends tokens) now runs on every call of that key, though a manual tier otherwise runs only when named by number.
    fix: "In markedScope, after the go.mod/no-marked checks are reachable, refuse a marked test whose tier is declared with manual: true and that the key's tiers do not list, through configRefusal naming the tests, the tier and keyName(gatesKey+\".\"+request.gate) to list it; leave every non-manual tier unrefused. Add one step to the marked case of TestGateTest_Scopes in internal/gatecli/scopes_integration_test.go: mark the fixture's tier as manual via scopesConfigWith, write a marked test in that tier, assert wantRefusal naming the tier and the key, then restore the config. Update the marked-scope paragraph of internal/gatecli/doc.go and the scope-marked line of internal/gateslot/template.yaml to say a manual tier's marked tests run only when the key lists that tier."
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/gatecli/scopes_integration_test.go:498
    summary: The rewritten case says the tier1 marked test runs under tier1's command, but wantArgs checks only invocation args and tier0 and tier1 share one stand-in command, so a run of the tier1 group under tier0's command or slots would still pass.
    fix: "After the wantArgs call at scopes_integration_test.go:498, keep the gate result in a variable and read guardRunRecordOf(t, g.stdout.String(), 1); assert GuardStep is true, Tier is 1 and Slots equals the pool size that tier1's slots: all resolves to (tier0 holds fewer), reporting the record on mismatch."
  - id: F3
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: _lym/landing/summary.md
    summary: The Changes section says internal/gatecli/cli.go states the new rule, but commit c368b94ac reverted the cli.go hunk and the range's diff leaves cli.go unchanged.
    fix: "In _lym/landing/summary.md, third bullet under ## Changes, drop internal/gatecli/cli.go from the list of files that state the new rule, leaving internal/gatecli/doc.go and internal/gateslot/template.yaml."
  - id: F4
    severity: NIT
    class: consistency
    origin: lens:code-correctness
    location: internal/gatecli/tiers.go:230
    summary: The undeclared-tier refusal, which doc.go and template.yaml now name as the refusal of a marked test of an undeclared tier, calls the refused tests guard tests even when they are marked tests.
    fix: "In planTierRuns' guard loop, word the configRefusal as \"the guard step runs the guard and marked tests of %s under its command, but the hub declares no %s\", keeping its args."
  - id: F5
    severity: NIT
    class: consistency
    origin: lens:code-affected-set
    location: pattern/PATTERN-test-speed.md:155
    summary: The upkeep bullet says the marked scope refuses a call that would leave a marked test out, which reads as the removed rule that the key's tiers must cover its marked tests.
    fix: "In pattern/PATTERN-test-speed.md line 155, replace \"and refuses a call that would leave a marked test out or run none.\" with \"each under its own file's tier whatever tiers the key lists, and refuses a call with a marked test of a tier the hub does not declare, or one that would run none.\""
---

### [MEDIUM] Marked tests of a manual tier now run unbidden

**Location:** `internal/gatecli/only.go#markedScope`

**Issue:** A manual tier (`gateslot.Tier.Manual`, "such as a tier that spends tokens") runs only when a call names it by number.
Before this diff a marked key reached a marked test's tier only when the key listed it, so a manual tier stayed opt-in.
The guard loop of `planTierRuns` (`internal/gatecli/tiers.go:224-249`) never reads `Manual`, and guard tests cannot sit in tier2/tier3 files (`cli.go:275` refusal), so marked tests are the one path that now runs a manual tier with nobody naming it.
The board entry asks that no hub need the key to reach its marked tests; it does not ask that a token-spending tier run on every upkeep call.

### [LOW] Tier1 case does not prove tier1's command and settings

**Location:** `internal/gatecli/scopes_integration_test.go:498`

**Issue:** The comment says the marked tier1 test runs "under tier1's command".
`wantArgs` compares only invocation args, and the fixture's tier0 and tier1 share `'<stand-in>' {targets} {args}` (`test_integration_test.go:640-641`).
The test shows a second guard group ran, not that it took tier1's command, env or slots.

### [LOW] Change description names cli.go as changed

**Location:** `_lym/landing/summary.md`

**Issue:** 1311d492f changed the `cli.go` help line, and c368b94ac reverted it; `git diff --stat` over the range lists four files and no `cli.go`, and `cli.go:75` still reads "Scope marked runs only the key's //lym:only tests."

### [NIT] Undeclared-tier refusal calls marked tests guard tests

**Location:** `internal/gatecli/tiers.go:230`

**Issue:** This diff routes a marked test of an undeclared tier to this refusal, and `doc.go:273` and `template.yaml` now point to it, yet its text reads "the guard tests of tierN".
The way forward, `lym config gate` from the prime to declare the tier, fits a marked key.

### [NIT] PATTERN-test-speed upkeep bullet still implies the removed rule

**Location:** `pattern/PATTERN-test-speed.md:155`

**Issue:** "refuses a call that would leave a marked test out" was written for the coverage refusal this diff removes; the one marked-test refusal left is a tier the hub does not declare.

## Rejected

- Merge danger door (handler): the diff changes no stored config value or record format, and a marked key's listed tiers ran no package before or after, so the `two-way` call and `gate` blast radius hold; a downgrade only brings back the old refusal.
- Scope: the template default stays `[0]`; the board entry offers deriving the tiers as an alternative to changing the default, and the diff takes that option (code-scope lens, no finding).
- Undeclared-tier refusal with no marked-scope test (handler): the `tiers_test.go` row "a guard group whose tier is undeclared" drives the same scope-independent branch, so the behaviour has coverage.
