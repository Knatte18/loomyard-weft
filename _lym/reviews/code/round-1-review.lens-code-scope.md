```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. Trace each change in the diff to the board entry's goal points, and each goal point to the changes that build it.

**Decisions.** The answer key is the decision record alone, and it has no `Added after Discussion` entries.
- **G1.** The batch gate's tier1 runs over the packages of the batch's own cards only, not their dependents.
- **G2.** Tier0, the card gate and the plan-level verify keep their scope.
- **G3.** About 14.6 min saved per Webster or Weaver row. This is an expected outcome, not something the code builds.

**Per change**
- **C1. Batch tiers narrowed to the cards' own packages, with the stderr notice.** This is `ownPackageTiers`, the `own` module list, `sc.tiers` filled per narrowed tier, and the `printLine` notice (`internal/gatecli/scope.go:242-269`, `:340-356`). Traces to G1, and its `request.gate != gateslot.BatchGateKey` guard and `number != 0` filter trace to G2.
- **C2. The guard step judges coverage per tier.** This is `scope.listedPackages` replacing `listedTiers`, the new `listedPackages`, the new `scanGuards` test, and `coveredTiers` deleted (`scope.go:84-87`, `:358-384`, `:683-703`). Forced by C1: without it, a dependent package's tier1 `//lym:guard` test runs in no step once tier1 stops listing dependents.
- **C3. Watch skip per share.** This is the `existingDirs`/`watchedSkip` extraction and `ownSkip` (`scope.go:251-262`, `:386-409`). Forced by C1: a narrowed share needs a skip list over its own packages, and the extraction gives the two call sites one implementation.
- **C4. `gateslot.BatchGateKey`, and `decodeGates` using it.** This is `internal/gateslot/tiers.go:127-128` and `internal/gateslot/config.go:521`. Forced by C1, which branches on the key name, and by PATTERN-code-conventions' named-constant rule.
- **C5. Docs.** These are `internal/gatecli/doc.go:154-157` and `:241-242`, `contracts/specs/loom-plan-spec.md:345-346`, and the `internal/gateslot/template.yaml:22` comment. Forced by C1 and C2, which change the behaviour these texts describe.
- **C6. New test row.** This is `internal/gatecli/scopes_integration_test.go:537-572`. Forced by C1, C2 and G2: it asserts the narrowed batch, the card key left unchanged, and the tier that covers tier0 kept whole.
- **C7. Commit c2662dece re-breaks comment lines.** Its edits fall in the comments C1 to C5 add, so it is part of those changes.
- **C8. Commit aad258757 drops `--no-carry` from the new test row.** Forced by C6.

**Per decision**
- **G1:** built by C1, with C2 to C6 forced by it.
- **G2:** built by C1's key guard and tier0 exclusion. The plan-level verify is untouched, since it is the `webster` key of scope all and never reaches the card branch. C6's `--gate card` assertion pins G2.
- **G3:** no change builds it, and none can. The summary labels it an estimate.

**Findings**
1. **NIT, scope.** Location: `internal/gatecli/scope.go:345-356`. `ownPackageTiers` narrows every kept batch tier above tier0 that does not cover tier0, not tier1 alone, while the entry names only tier1. On a hub whose batch key lists tier2 or above, the diff also drops the dependents from those tiers, and the entry neither measures nor asks for that. Evidence: `scope.go:351` (`number != 0 && !slices.Contains(tier.Covers, 0)`) against the board entry's `## Change` in `_lym/discussion/decision-record.md:12`. The generalisation is defensible, since it avoids hard-coding a tier number and follows the entry's reasoning that slower tiers of the dependents run once per plan. It is flagged only so the parent can confirm the wider reach.
2. **No finding.** The diff departs from the entry's literal text on a hub where tier1 covers tier0: there tier1 keeps the dependents (`scope.go:351`). G2 forces this, because narrowing a tier that covers tier0 would narrow tier0's tests too, so the diff settles the conflict between the two goal points in favour of G2. The summary's Design decisions section states the choice.

Tool calls: none.
