```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge each reviewer proposal the fixer carried out (fix-proposal lens).

**LOW: card 3, the changed D9 sentence "try one only in another hub on the machine, on lym's own repository the sandbox hub;"**
- Summary: The proposal removes the only pointer to the sandbox hub and puts nothing in its place, so the step may not be possible to carry out.
- Evidence: Take an agent running a sampler in a repository other than lym's. It reads "the sandbox hub" and has no path, marker or command that tells it where that hub is or how to find it. The new sentence in the card says `docs/sandbox-howto.md` is something only the writer reads, so the stencil gives no way to find the hub. Removing the path is correct. A better fix would also say how the agent finds the hub, for example by naming it as the hub the operator points it to, or by having the agent ask in the pane. The wording "on lym's own repository the sandbox hub" also reads as a broken phrase now that the clause ending in "describes" is gone.

I challenged the other proposals and they hold:
- **F1:** `"SamplerSession"` sorts between `"RecordWrite"` and `"SeedChild"`, because "Sa" comes before "Se".
- **F2:** The sampler recipe sets no `amendment_target`, so `AmendmentTarget("sampler")` returns "" and the widened doc stays true.
- **F3:** `childSeedParams` returns `params.parent` for any recipe `childRecipeSupported` admits, and `seedAndCommitBootstrap` writes the parent through `loomSeedFor` for each loom seed, so the doc claim holds.
- **F5, card 8:** Both production calls of `codeBudgetZero` and the one call of `rerouteReviewed` are in `landingDeps` (landingdeps.go:82 and :272) and in `rerouteReviewed` (:393), as the rewritten summary says.

Tool calls:
- awk/grep over `internal/loomcli/landingdeps.go` and `internal/battencli/*.go`: to find which function holds each seam call and where HandBack is wired. My context had only the line numbers from grep, not the functions around them.
