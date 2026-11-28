```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens. For each function and branch the diff adds or changes, run one concrete case through it.

**Cases run**

- `computeScope` card branch, `--gate card` with tiers [0,1] and no covers: `ownPackageTiers` returns nil, so `sc.tiers` stays nil, no notice prints, and `forTier` returns the whole dependents scope. Correct, the same as before the diff.
- `computeScope` card branch, `--gate batch` with tiers [0,1] and no covers: `own` = the cards' packages, `sc.tiers[1]` = {own, ownSkip}, tier0 keeps `modules`, and the notice prints. Correct.
- `computeScope` card branch, `--gate batch` where tier1 covers tier0: `keptTiers` = [1]. `ownPackageTiers` drops tier1 because it covers 0, so nothing is narrowed. Correct.
- `computeScope`, `--gate batch` listing only [0]: `ownPackageTiers` is empty and the scope is unchanged. Correct.
- `computeScope`, `--gate batch` where one card target is undeclared or the tier is undeclared: `ownPackageTiers` skips a tier whose `cfg.Tier` fails, and `planTierRuns` then refuses. Correct.
- `computeScope`, cards with no Go packages and no `--main-packages`: `modules` and `own` are both empty, so tier0 and tier1 each run nothing, as before. The notice still prints, which is harmless. Correct.
- `computeScope` with `--main-packages`: `own[i]` is cloned from `module.Packages` before either `withMainPackages` call, so appending to `modules` cannot alias `own` (scope.go:245–248). Both sets receive the mains. Correct.
- `computeScope` when `cardDependents` fails (`go list` error): `modules` and `own` hold the same packages and only the tier shares differ. Correct.
- `computeScope` with `changesRead` false: `sc.skip` and `ownSkip` are both nil, so nothing is skipped. Correct.
- `computeScope` with `changesRead` true and a watch scan error in the own directories: the refusal is returned through `watchedSkip`, the same refusal text as before. Correct.
- `existingDirs` / `watchedSkip`: same logic as the inline code they replace. A package the card has yet to create is still dropped. Correct.
- `ownPackageTiers` for a non-batch key or an empty key: returns nil. Correct.
- `listedPackages`, card scope with nil `sc.tiers`: each kept tier and every tier it covers maps to the guard-module packages of `modules`. This equals the old `inside` × `listedTiers` check. Correct.
- `listedPackages`, batch case: [0] maps to the dependents and [1] to the own packages. A dependent's tier1 guard goes to the guard step, and an own package's tier1 guard does not. Correct.
- `listedPackages` with a card module other than `guardDir`: that module is skipped, as before. `path.Clean("")` gives "." when `guardDir` is empty. Correct.
- `scanGuards` for the computed, `--base`, targets and `--guards` forms: `listedPackages` is nil, so every guard counts as left out. The old code did the same (`inside` was empty because `modules` was nil). Correct.
- `scanGuards`, a tier0 guard in a dependent under batch: it is in `listedPackages[0]`, so the guard step leaves it out. Correct.
- `forTier`, `planTierRuns`, `scopeInvocations` and `ranWhole` for narrowed tier1 when the tier has `carry: false`, the call passes `--no-carry`, or the tier's command holds no `{targets}`: **wrong**. See finding 1.
- `planTierRuns` watch skip for the narrowed tier: it uses `share.skip` = `ownSkip`, a subset of `sc.skip`. `holdsTierTest` gets the own-package names. Correct.
- Vet step under batch: it plans over `sc.modules` (the dependents) for tag tiers no listed tier is or covers. A listed narrowed tier1 gets no vet run. That matches what the doc says about the vet step and does not contradict the board entry. Correct.
- `decodeGates` using `BatchGateKey`: the same value as the old literal. Correct.

**Findings**

1. LOW, design. Location: `internal/gatecli/scope.go:263-270`. When a batch tier runs whole because of `carry: false` or `--no-carry`, or because its command lacks `{targets}`, the call still reports and plans it as narrowed.
   - `scopeInvocations` runs that tier whole (`case sc.whole, tier.NoCarry` at `internal/gatecli/tiers.go:299`, and `!gateslot.HoldsTargets` at tiers.go:307).
   - `computeScope` still prints "runs tier1 over the cards' packages alone" (scope.go:268).
   - `listedPackages` still records only the own packages for that tier (scope.go:368).
   - `ranWhole` reads `sc.forTier(1).whole`, which is false (tiers.go:411).
   - So the guard step reruns every dependent package's tier1 `//lym:guard` test that the whole tier1 run has just run, and the stderr line misstates what ran.
   - Before the diff, those dependents' tier1 guards counted as covered (scope.go diff, old `inside`/`listedTiers` check), so this redundancy is new. Commit aad258757's message confirms that `--no-carry` runs every tier whole.
   - Fix: leave a tier out of `ownPackageTiers` when it runs whole, meaning its `NoCarry` is set (`carry: false`, or `request.noCarry`) or its command has no target placeholder.

No tool calls made.
