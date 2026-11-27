```yaml
lens: code-affected-set
outcome: reported
```

**Per change — affected set**

1. `markedScope` no longer refuses unlisted-tier marked tests; `markedNames` deleted (`internal/gatecli/only.go`).
   - Callers: `computeScope`/`isMarkedCall` (scope.go:126,183), `withOnlyTests` (only.go:48) — covered; `markedNames` had no other caller.
   - Downstream: `planTierRuns` guard groups, `ranWhole`, `notRunTiers`/`withOtherTiers`, `tierNumbers`, `runGuard`/`guardStepRun`, run records, green — all operate on `sc.guards`, none relies on listed tiers covering them. Covered.
   - Board-tidy prompt renders the upkeep call via `AgentGateCall` (beamcli/dobbydeps.go:420), independent of the refusal. Covered.
   - Stencils `beam-fragment-board-tidy-upkeep.md`, `beam-template-board-tidy.md` only embed the rendered call; they state no tier rule. Covered.
   - Test asserting the refusal rewritten (scopes_integration_test.go:493-502). Covered.
   - Repo-wide search for the old refusal text ("lists no tier that", "cannot run:", "gates.upkeep") — no stale occurrence. Covered.
2. Marked test of an undeclared tier now reaches the guard step's refusal (tiers.go:230) — the refusal's own wording speaks of "guard tests"; left slightly off (NIT below).
3. Doc text — `internal/gatecli/doc.go`, `internal/gateslot/template.yaml` updated. `cli.go` help (cli.go:75) left as at base — fine for code, but the change description claims otherwise (LOW below). `pattern/PATTERN-test-speed.md:154-155` — still the old shape (NIT below). `pattern/PATTERN-test-only-code.md:30-31` — only says the upkeep run runs the dead-code guard; covered.
4. Commit messages — `Root-Cause:` trailer on 1311d492f names the defect the diff removes (the refusal in `markedScope`); holds. No `Pattern-Departure:` or `Loosened-Check:` trailers; the refusal assertion was removed because the behaviour is intentionally removed, not loosened. No declared PATTERN changes in the directive, so nothing to land.

**Findings**

- LOW, consistency — `_lym/landing/summary.md` (Changes, third bullet): it says `internal/gatecli/cli.go` states the new rule, but the range's net diff leaves cli.go unchanged — commit c368b94ac reverts 1311d492f's cli.go hunk, `git diff --stat bae1e3af7..HEAD` lists only doc.go, only.go, scopes_integration_test.go and template.yaml, and cli.go:75 reads "Scope marked runs only the key's //lym:only tests."
- NIT, consistency — `pattern/PATTERN-test-speed.md:154-155`: "refuses a call that would leave a marked test out or run none" still implies the removed rule that a key's tiers must cover its marked tests; now the only marked-test refusal left is an undeclared tier. Evidence: pattern/PATTERN-test-speed.md:154-155; only.go:82-106.
- NIT, consistency — `internal/gatecli/tiers.go:230`: refusal "the guard tests of %s run under its command, but the hub declares no %s" is now the refusal doc.go:273 and template.yaml:20 point to for marked tests, yet it calls them "guard tests" even when they are `//lym:only` marked tests. Evidence: tiers.go:230; doc.go:273.

**Tool calls**

1. Repo-wide grep for old refusal text and "gates.upkeep" — to check for stale references; context only partially covered this.
2. List non-doc files mentioning upkeep — to find stencils/specs describing the rule.
3. Grep the upkeep stencils' tier lines — to check whether stencils state the tier rule.
4. Grep `UpkeepGateKey` callers and `upkeep_call` rendering — to follow the prompt's upkeep call one hop out.
5. Read `AgentGateCall` in gateslot/tiers.go — to confirm the rendered call does not depend on the removed refusal.
