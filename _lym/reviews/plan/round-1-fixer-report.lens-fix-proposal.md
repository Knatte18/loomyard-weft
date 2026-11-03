```yaml
lens: fix-proposal
outcome: reported
```

Task: test the reviewer's proposals the fixer carried out as proposed (F1, F2, F3, F5, F6, F7), looking for cases where a proposal is wrong or breaks something.

1. **MEDIUM. F2, card 4's new line "`shed-recipe-spec.md` gains an expansion paragraph naming `Expand`, the told resolver, the block exit and the block path."**
   The new paragraph will contradict the spec's own "Validation split" paragraph, and the proposal leaves that paragraph as it is.
   Evidence: `contracts/specs/shed-recipe-spec.md:112-113` says the package "runs no reachability, cycle, blind-gate, dangling-target, or segment analysis of its own".
   Card 4's `Expand` does three of those: it refuses a block recipe that includes itself (a cycle), refuses an inlined `on_done`, `on_stuck` or `on_reroute` that names no row of its recipe (a dangling target), and refuses a shared segment (segment analysis).
   Card 4 also needs to say that the Validation split paragraph names `Expand`'s block refusals as the exception.

2. **LOW. F2, the same card 4 spec line.**
   The proposal adds `Expand` to the spec but leaves the "**Four exported functions.**" paragraph (`shed-recipe-spec.md:93-96`) unchanged, so that paragraph no longer lists every exported function.
   It already omits `RoutingOf` and `NewShed`, but a paragraph that names `Expand` beside a heading that says four will be noticed.
   The card should also reword that heading or extend its list.

No breakage found for the other proposals:
- **F1:** `Build` runs its unexpanded-row check only when a row carries `Block` and its engine is not `Block`, and the every-engine test's row leaves `Block` empty.
- **F3:** `TestInterruptPolicies_MatchEngineIdentity` iterates `rowEngines()`, which comes from tapestry's `wantProducerTable`, so it never sees Motif. `TestInterruptPolicies_ClosedVocabulary` accepts `InterruptPolicyReinvoke`.
- **F5, F6, F7:** nothing to report.

Tool calls:
- One grep for tests besides `TestInterruptPolicies_MatchAssembledRows` that pin `InterruptPolicies`, and for `rowEngines`' source. My context held only `interruptpolicy_meta_test.go`.
