```yaml
lens: code-affected-set
outcome: reported
```

Lens code-affected-set: what the diff's changes force outside the lines they touch.

**Affected set per change**

1. The `--card` scope widens to reverse dependents (`scope.go` `computeScope` and `cardDependents`).
   - `scanGuards` (`scope.go:571-603`) now treats every dependent as inside, `cmd/lym` included. **Not covered** (F1).
   - `PATTERN-test-speed.md:120`, the guard-step rule. **Left wrong** (F1).
   - `gatecli/doc.go:224`, "keeps full guard coverage". **Left wrong** (F1).
   - `tagTiers` and the vet step (`tiers.go:165`): they compute over the widened set, and the test row expects vet `./lib ./user`.
     The prose in `gatecli/doc.go:211-212` still says "card package" and "The run covers the card's packages". **Stale** (F3).
   - `gatecli/doc.go:134`, "takes the packages of the named cards": line 140 completes it. Covered.
   - Watched-test skip (`FindWatchedTests` over `dirs`): computes over the widened set. Covered; `doc.go:241-243` wording still holds.
   - `--map --card` (`testmap.go:202`, `candidatePackages`): picks up the widened modules automatically.
     The comment at `testmap_integration_test.go:735` still holds for its single-package fixture. Covered.
   - Other card-scope statements are updated in the diff: the `--card` flag help (`cli.go`), `template.yaml:25`, `loom-plan-spec.md:327,335`, `docs/overview.md:960` and `impactset/doc.go`. Covered.
   - `PATTERN-test-speed.md:50`, the gate-key table, states no package set. Covered.
   - `webster-body-implementer.md:73-76`, the guard-step deviation rule: the wording changed, but its consequence did not. **Left inconsistent** (F2).
   - `_lym/landing/summary.md`, "the guard step all work from the widened set": it describes the change but not what that does to guard coverage. Part of F1.
2. The new `impactset.DependentPackages` and `dependentsFor`: the only caller is `cardDependents`, and `impactset/doc.go` is updated. Covered.
3. The plan rules (new function rides with its first caller; repository-wide intent names its search): `loom-block-plan-cards.md`, `loom-rubric-plan-review.md` and `loom-plan-spec.md` all carry both rules. Covered.
   The plan-checks spec states no granularity rule, so it is not touched.
4. Test expectation rows in `scopes_integration_test.go` changed from `./lib` to `./lib ./user`.
   This is a stricter exact-equality want, not a loosened check, so no `Loosened-Check:` trailer is needed. Covered.
5. Commit messages: `e941635d0` carries a `Root-Cause:` trailer naming the `computeScope` `--card` branch, which the diff touches. No `Pattern-Departure:` trailers.
   The directive declares no PATTERN changes, so nothing to check landed. Covered.

**Findings**

- **F1. BLOCKING, design.** Location: `internal/gatecli/scope.go:198` (the `cardDependents` call), with `scanGuards` at `scope.go:577-590`.
  The widened card scope pulls `cmd/lym` into nearly every card's scope, so its `tier1` `//lym:guard` tests stop running at the card gate. This breaks `PATTERN-test-speed`'s guard-step rule.
  - Evidence:
    - `go list -deps ./cmd/lym` reaches 188 internal packages, among them `planparser`, `shedrecipe`, `impactset` and `gatecli`. So `cmd/lym` lands in `DependentPackages` for almost any card.
    - `scanGuards` marks every scope target as inside and drops its guards from the guard step, whatever their tier (`scope.go:582-590`).
    - The `card` key lists tier 0 only (`internal/gateslot/template.yaml:35`, `card: {tiers: [0], ...}`), and the hub's `gate.yaml` does not override it.
    - `cmd/lym` holds three `tier1` guards, `TestTestOnlyCode_EveryProductionFunctionIsReachableFromAMain` among them (`cmd/lym/testonlycode_linux_test.go:79-81`). The listed tier0 run skips them and the guard step leaves them out.
    - `pattern/PATTERN-test-speed.md:120` says "A narrowed gate's guard step runs every marked test by name under its file's tier, whatever the impacted set". `internal/gatecli/doc.go:224` says the step "keeps full guard coverage". Both are now false for the card gate.
    - It also removes, silently, the very guard issue #262 reports firing at the card gate.
    - The summary's claim that "the guard step all work from the widened set" hides this.
  - Fix: in `scanGuards`, count a guard test as inside only when the listed tiers run its tier, or keep guard tests of a tier the call does not run in the guard step. Add a row where a dependent package holds a guard of an unlisted tier.
- **F2. MEDIUM, consistency.** Location: `contracts/stencils/webster/webster-body-implementer.md:73-76`.
  The stencil still says the guard step is what fails on a `cmd/lym` pinned set, and sanctions changing "only a file the guard step's failure names".
  With `cmd/lym` now inside the card scope, its tier0 guards run in the listed tier0 run, not the guard step. Such a failure then falls outside the sanctioned deviation the stencil states.
  Evidence: the stencil at line 73 against `scanGuards` at `scope.go:582-590`.
  Fix: have the stencil cover a guard failing in the listed tier's run over a dependent package too.
- **F3. LOW, consistency.** Location: `internal/gatecli/doc.go:211-212`.
  The vet-step prose still says "a card package" and "The run covers the card's packages". The vet now runs over the widened set, as the changed row `[]string{"vet", "-tags", "tier1", "./lib", "./user"}` shows (`scopes_integration_test.go:493`).
  Fix: say "the card scope's packages".

**Tool calls**

1. A grep of rule and package docs for card-scope statements: its output was too large and persisted, so I re-ran it narrower.
2. The narrowed grep of `PATTERN.md`, `pattern/`, the `gateslot` and `gatecli` docs, the specs and the stencils for card-scope wording: the manifest held no repository-wide search of the rule docs.
3. Reads of `gatecli/doc.go:130-136,236-246`, `loom-plan-spec.md:336-346` and the guard lines of `PATTERN-test-speed`: these doc spans were not in the manifest.
4. A grep of `gateslot/doc.go` for card-scope lines, plus `PATTERN-test-speed.md:116-124`: needed the guard rule's full text and the `gateslot` doc, which the manifest lacked.
