```yaml
lens: leanness
outcome: reported
```

Leanness lens: what in the decision record to move or cut.

**Findings**

- **MEDIUM**: Constraints, first bullet ("The cancel path's remote deletions run synchronously …").
  - The bullet carries a research narrative: what the board guessed, and that the guess does not match the code. That narrative belongs in the support log.
  - Keep only the requirement the plan writer needs: "Cancel's remote deletions are synchronous; its spawned push is the prime's records push and deletes no branch."
  - The clause "the board's guess … does not match the code" is deliberation. It also repeats Open risks bullet 1.

- **MEDIUM**: Open risks, bullet 1, and the Constraints bullet above say the same thing twice. Both state that no background work of the cancel deletes a branch.
  - Say it in one place: the fact in Constraints, the unpinned trigger in Open risks.

- **MEDIUM**: The rationale paragraphs in D1, D3, D4 and D7 restate their decision or describe today's code.
  - D1's "this is the board's own workaround …" is support-log narrative.
  - D3's rationale ("with the pair gone it runs no reached-parent refusal …") carries a claim the plan writer needs. Move that claim into the Decision body or Constraints and cut the rest.
  - D7's rationale restates the decision.
  - Shorten each to one clause.

- **LOW**: Constraints bullet "`PATTERN-pair-teardown`: … D1 re-creates a pair and removes none". This is a non-interaction note with no requirement in it. Cut it.

- **LOW**: Constraints bullet on `fabric-remote-async`. Scope already excludes that note. This bullet only justifies D4's synchronous push, so it belongs in D4's Bound, which already says `Remove` waits on one more push. Cut it from Constraints.

- **LOW**: Auto-mode assumptions repeats picks that the Decisions already state (D1's rows, D2's `failed`, D3's choice, D7's reuse of the lock). The support log's question ledger already records them as auto-picks. Reduce this section to the assumptions not stated in a Decision, or to a single line pointing at the ledger.

- **LOW**: Notes for the plan writer, bullets 1 and 3, duplicate entries in `relevant.md`: the `owner.go` location and the `destroy.go` lease executor. Bullet 2, about reusing the coalesced push in-process, is a real hint. Keep it.

- **LOW**: Emphasis.
  - D3 is long for what it does, which is mostly stop text.
  - D3 claims that cancel runs no reached-parent refusal once the pair is gone. That claim rests on `childFound` being false. The record carries no evidence for it, while Open risks bullet 2 hedges the same point.
  - Either state the evidence (`internal/battencli/cancel.go` `childReachedParent` returns false with no child status unless batten stands at Land or Worktree-Teardown) or drop the claim from the rationale.

**Tool calls**: none. My context held everything this lens needed.
