```yaml
lens: fix-consistency
outcome: reported
```

Task: consistency check of the round-1 fix change set against its surroundings.

1. **MEDIUM**: `internal/gatecli/cli.go` help, "also run the //lym:guard tests the tiers leave out"
   - The new wording is false for the computed and `--base` scopes. The same sentence covers them ("a narrower scope while scoper is go").
   - There the guard step takes `targets.Guards`, which holds only the guards of packages *outside* the impacted set, whatever their tier (`internal/impactset/doc.go:37`).
   - So a tier1 guard in an impacted package is left out by a tier0 call and is not run by the guard step either.
   - Only `--card` got the tier-aware rule (`scanGuards` with `listedTiers`).
   - The old text, "outside the scope", was true for computed and `--base`. The new text should keep it and name the `--card` addition within the 2000-byte cap, or say nothing about `--card`.

2. **MEDIUM**: `_lym/landing/summary.md` (the change description) was left stale, although F1's `fix` explicitly asked for its guard-step sentence to be updated.
   - "The vet step, the watched-test skip and the guard step all work from the widened set" does not state the new unlisted-tier rule.
   - "the gate logs the cause" omits the new stderr line (F2).
   - Its Tests section says `TestDependentPackages` and `TestCardDependents` "cover an importer and a tier-tagged test importer being added, and the failure path". `TestCardDependents` is now `TestCardDependents_UnlistableModuleKeepsTheCardsPackagesAndSaysSo` and covers the failure path only.
   - It names neither the new `TestGateTest_Scopes` guard row nor the `--map --card` importer assertion.
   - If the fixer may not write `_lym`, the fixer-report has to say this part of F1 was left undone.

Outside the change set, one sentence only: `doc.go`'s "A guard test's tier comes from its file's `tierN` build tag" disagrees with `impactset.FileTier`, which takes the lowest `tiergate.Require` first. The new `--card` sentence therefore does not cover a higher-tier guard in a mixed-tier file. This predates the fix.

I found nothing else stale: the plan spec's per-card bullet, `docs/overview.md`, the implementer stencil's lines 45 and 73-77, and the `DependentPackages` doc all still agree with the code.

Tool calls (3 of 6):
- `git grep` for guard-step wording across contracts, docs and impactset: my context lacked the twins outside `internal/gatecli`.
- `git grep` for "outside the scope" and card-package comments in gatecli and impactset: my context lacked a post-edit sweep of code comments.
- `git grep` plus `sed -n 328,340p contracts/specs/loom-plan-spec.md`: my context lacked the plan spec's card-gate sentences.
