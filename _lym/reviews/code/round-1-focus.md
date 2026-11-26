---
round: 1
focus:
  - "Guard step against the widened scope: `scanGuards` in `internal/gatecli/scope.go` treats every package of the card scope as inside, whatever tier its guard test carries. Settle whether a `//lym:guard` test in a newly added dependent package (`cmd/lym` especially) can now run in neither the listed tiers nor the guard step, for a gate key that lists a lower tier than the guard's."
  - "Whether every package `DependentPackages` adds can be handed to each listed tier's command and each vet run: packages whose files are all behind a build tag or platform constraint, test-only packages, and what `listPackages` does with a package that fails to load. Compare with how the diff scope filters the same graph's output, and check `tagTiers`, `FindWatchedTests` and the skip list over the larger set."
  - "Whether the widening reaches the cases the board entry reports. Check against the real module that the named failing packages depend on a package the card would have named, including the `internal/shedrecipe` case, and test the change description's claim to that effect. Check also a card whose targets are non-Go files that a package embeds or reads, and whether the same graph rules as the diff scope apply to them."
  - "The failure path of `cardDependents`: a `go list` that fails or is cut off by `ctx` narrows the gate to the cards' packages and only calls `logger.Warn`. Settle whether the caller sees this, whether a gate call can then pass over a narrower scope than documented, and whether a partial listing can yield a silently incomplete set."
  - "Other consumers of the card scope: `--map --card` (`internal/gatecli/testmap.go`), `--main-packages` merging and de-duplication, multiple cards, a module directory other than the root, and path forms (`.`, `./x`, trailing or unclean paths) between `CardPackages` and `dependentsFor`."
  - "Statements of the card scope that the diff may have left saying 'the cards' packages' alone: the `gate test` command help, the rest of `internal/gatecli/doc.go`, the loom specs, stencils and refusal or finding texts. Search the repository rather than only reading the changed hunks."
  - "The two plan rules in `loom-block-plan-cards.md`, `loom-rubric-plan-review.md` and `loom-plan-spec.md`: read each as the diff leaves it for agreement among the three, for conflict with the neighbouring granularity, Delete and call-site rules, and for cases the new-function exception makes unsatisfiable or leaves undefined."
  - "Test coverage of the behaviour the widening changes downstream (vet tags, watched-test skips, guard step, `--map --card`), and overlap among `TestDependentsFor`, `TestDependentPackages`, `TestCardDependents` and the changed `TestGateTest_Scopes` rows."
---

Seed focus for round 1, drawn from the board entry, the change description and the diff over the review range.
No round has been judged, so no entry states a verdict.
