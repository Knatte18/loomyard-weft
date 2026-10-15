# Card 2 — Fork prefix and FRESH-READ fragments: no code reaches the fork

**Edit:**
- `contracts/stencils/webster/webster-prefix-fork.md`
- `contracts/stencils/webster/webster-fragment-fresh-read-plain.md`
- `contracts/stencils/webster/webster-fragment-fresh-read-pack.md`
- `contracts/stencils/webster/webster-fragment-fresh-read-pack-card.md`
- `contracts/stencils#wordingClaims`
- `internal/websterengine_test#prePackStepOne`
- `internal/websterengine_test#prePackFreshRead`
- `internal/websterengine_test#writePrePackBody`
- `internal/websterengine_test#TestRenderPrompts`

**Uses:**
- `contracts/stencils/webster/webster-fragment-fresh-read-pack-inline.md`

**Intent:** Carry decisions D2 and D3: the fork prompt stops claiming a codebase orientation reaches the fork from Master.

In `webster-prefix-fork.md`:

- The banner comment's "it inherits Master's whole context already: the codebase orientation, the plan's framing, and every constraint Master already read up front" becomes "it inherits Master's whole context already: the plan's framing and the repo's rules Master read up front, and no code, since Master never reads code".
- The title becomes `# Webster fork implementer — one batch of cards, forked from Master`.
- The opening paragraph's second sentence becomes: "You inherit from Master the plan's framing and the repo's rules, and no code, so this prompt is deliberately thin: every file your cards need you read yourself, as the card steps below say."
- `## You are the IMPLEMENTER, not the driver — never run `lyx webster`` and `## Merriam's contract files are not yours` are unchanged, and the prefix keeps `{{.outcome_path}}`, `{{.summary_path}}` and `{{.friction_directive}}`.

In each of the three live FRESH-READ fragments, the sentence "Inherited context can be stale.", the sentence "A file you looked at during orientation — your own, or one Master read before forking you — is not necessarily the version on disk right now: …" and the sentence pair "Only your own reads, taken now, are current;\ncontent you merely inherited is not." are replaced by one statement, the same two lines in each fragment, placed where the first sentence stood:

```
No code reaches you from Master: the plan and the repo's rules you hold stay current for this run, except a rules file a card of this run targets, which you read fresh like code.
Only your own reads of code, taken in this batch, are current.
```

In `webster-fragment-fresh-read-plain.md` the remaining line's "re-read — in THIS turn —" becomes "read — in THIS turn —", so the rule holds for the cold recovery strand that renders this fragment and inherits nothing.
Every other line of each fragment is unchanged, and each banner stays.
`webster-fragment-fresh-read-pack-inline.md` is not touched: its banner makes it the frozen layout baseline.

In `contracts/stencils#wordingClaims`, every new pin is a short phrase, never a whole sentence:

- The `webster-fragment-fresh-read-plain.md` row replaces its three old phrases with "No code reaches you from Master", "except a rules file a card of this run targets", "Before you edit anything, read — in THIS turn —" and "Only your own reads of code, taken in this batch, are current.", and its why becomes: a fork without a pack, and every recovery strand, reads every card file and target file in the same turn, and no code reaches it from Master.
- The `webster-fragment-fresh-read-pack.md` row drops the groups pinning "Inherited context can be stale.", "is not necessarily the version on disk right now: …" and "Only your own reads, taken now, are current;", and pins "No code reaches you from Master", "except a rules file a card of this run targets" and "Only your own reads of code, taken in this batch, are current." instead.
- The `webster-fragment-fresh-read-pack-card.md` row gains the same three phrases.
- Each of those three rows gains a `wantNone` of "one Master read before forking you" and "Inherited context can be stale.".
- The `webster-prefix-fork.md` row gains a `wantAll` of "the plan's framing and the repo's rules, and no code" and "every file your cards need you read yourself", and a `wantNone` of "codebase orientation" and "inheriting Master's context".
- The `webster-fragment-fresh-read-pack-inline.md` row is unchanged.

`internal/websterengine_test#prePackFreshRead` keeps its name and holds the new plain fragment's bytes after `"## The FRESH-READ rule\n\n"`: the two statement lines, each ending `\n`, then the "Before you edit anything, read — in THIS turn — … list name." line ending `\n`, then `"\n## Prior-batch context"`, so a blank line still separates the fragment from the next heading as in the current constant.
The comment above the `const` block shared by `internal/websterengine_test#prePackStepOne` and `prePackFreshRead` says they hold the implementer body's step 1 and FRESH-READ rule in their pack-off texts, so a pack-off prompt is pinned to those bytes rather than to the fragments that carry them; `prePackStepOne`'s value is unchanged.
`internal/websterengine_test#writePrePackBody`'s comment says it rewrites the body into its pack-off form (the pack-off texts in place of the markers, and no pack marker) instead of "the body as it stood before the pack"; its code is unchanged.

In `internal/websterengine_test#TestRenderPrompts`:

- The row "fork with a nil pack locates by quarry and renders the pre-pack bytes" is renamed "fork with a nil pack locates by quarry and renders the pack-off bytes"; it pins the pack-off fork prompt to the new `prePackFreshRead`, and the row "recovery locates by quarry and carries no pack section" already pins the recovery prompt to the same constant.
- The row "fork with a resolved multipart pack lists each declaration and locates by the pack" changes its `notContains` needle "Before you edit anything, re-read" to "Before you edit anything, read — in THIS turn —", so it still fails if the plain fragment leaks into a pack-on prompt.
- The rows "fork with a resolved multipart pack lists each declaration and locates by the pack" (file layout) and "fork with a per-card pack reads the card files first and fetches each card's pack" (per-card layout) each gain `contains` "No code reaches you from Master", and the row "fork carries the card gates and no leftover marker" gains `contains` "the plan's framing and the repo's rules, and no code".
  These three render needles are kept on purpose: acceptance criterion 4 names the rendered fork prompt in the per-card, file and pack-off layouts; the `notContains` side stays with the claim rows alone.
- The inline row "fork with an inline pack carries the 1e39afc10^ texts and the pack before the sandbox rule" and `inlineFreshReadBody` stay unchanged and keep pinning the frozen inline text.

**ImpactSummary:** Four stencils and their claim rows, three test constants or helpers' comments and values, and five render rows in one test; the inline fragment and every marker stay as they are, and no Go production code changes.

**Commit:** 2: webster: the fork prompt says no code reaches it from Master
