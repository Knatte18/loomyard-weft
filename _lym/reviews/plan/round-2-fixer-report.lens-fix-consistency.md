```yaml
lens: fix-consistency
outcome: reported
```

fix-consistency lens: I checked whether the changed text still agrees with sibling cards, decisions and cross-references.

LOW — `_lym/plan/03-sampler-stencil.md` Intent. The added sentence says "the stencil carries no `docs/`, `contracts/` or `internal/` path", and the D9 sentence no longer names `docs/sandbox-howto.md`. The decision this card cites still ties the rule to that path, so the two now read differently.
Evidence: `_lym/discussion/decision-record.md:156` says "try it only in another hub on the machine, on lym's own repository the sandbox hub that `docs/sandbox-howto.md` describes". Card 3's stencil wording no longer carries that path, so a reader comparing card 3 with D9 sees a mismatch. The decision record is outside the fixer's write surface (`_lym/plan` only), and D9 states the rule rather than the stencil's text, so this is a cross-reference a reader would notice, not a contradiction in behaviour. The fixer-report could name it, or card 3 could state that D9's path reference is the writer's source, which `**Uses:**` already records.

I found nothing else. These checks all agree:
- Card 5's ImpactSummary ("`loomrecipe.verifyGatedRows`, which card 6 narrows") matches card 6's Intent and Edit list.
- Card 6's ImpactSummary `InterruptPolicies` entry matches its Intent and its `internal/loomshed#InterruptPolicies` target.
- Card 8's "one test covers both" matches its Intent, where both seams' rows sit in `TestRerouteReviewed_ReadsTheCodeBudgetFromLoomYaml`.
- Card 11's `loomcli/doc.go` mention matches its Edit list.
- Card 7's new targets and Intent sentence don't collide with its "Comments that … stay" list.
- No other card names `TestNames`, `AmendmentTarget`, `childSeedParams` or `InnerRunDeps`.

Tool calls:
- One grep over `_lym` for sibling references to the changed symbols and to `docs/sandbox-howto.md`. My context held only the seven cards, not the decision record, the discussion files or the other cards.
