---
round: 1
focus:
  - "internal/gatecli/scope.go computeScope: follow the new per-tier `sc.tiers` entries of a card scope through every reader of the scope (`forTier`, the run planning, carry and `--no-carry` or `carry: false` whole runs, the vet step, the empty-scope path, skip and report lines). Does each behave correctly for a card-form scope that now has per-tier shares?"
  - "internal/gatecli/scope.go listedPackages and scanGuards: is every `//lym:guard` test run by either a listed run or the guard step, and not needlessly by both? Cover covered tiers, tiers dropped by keptTiers, a listed tier run whole, a guard module other than a card's module, and a failed or partial `go list`."
  - "internal/gatecli/scope.go ownPackageTiers: does the narrowing match the board entry, that only the batch gate's higher tiers lose the dependents, while tier0, the card gate and the plan-level verify keep their scope? Check calls with `--tier`, an unlisted or covering tier, and whether Webster and Weaver batch calls both reach this path."
  - "internal/gatecli/scope.go watch skips: are `ownSkip` and `sc.skip` right for each tier's package set, including when the card changes cannot be read?"
  - "internal/gatecli/scopes_integration_test.go: which behaviors the diff adds have no coverage, for example main packages in a narrowed tier, watch skips of a narrowed tier, or a card naming a package not yet created? Check existing tests before calling any of them uncovered."
  - "Docs against code: read internal/gatecli/doc.go, contracts/specs/loom-plan-spec.md, the template.yaml comment and the new Go doc comments against the behavior (stream the notice goes to, which tiers are narrowed, guard step rule). Does any other text in the repository still describe the batch gate's scope the old way, or quote a figure measured on it?"
  - "_lym/landing/summary.md: check its figures against docs/research/webster/gate-time-2026-11-28.md, its claims against the diff, its wording against the rubric's `Names of the tooling` bullet (for example the Design decisions section's reference to what 'the record asks for'), and its `two-way` door and blast radius against `Merge danger`."
---
Seed focus for a decision-record-only review.
The answer key is the board entry under `## Goal`; the record holds no added entries.
The risk is concentrated in `internal/gatecli/scope.go`, where a card scope gains per-tier shares and the guard step's coverage test changes.
