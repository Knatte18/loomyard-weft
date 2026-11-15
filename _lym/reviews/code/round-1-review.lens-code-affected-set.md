```yaml
lens: code-affected-set
outcome: reported
```

Lens code-affected-set: what the diff's two stencil changes force outside the lines they touch.

**Per change, the set found and whether it is covered**

- **Agent stencil Step 3 gains three bullets** (`contracts/stencils/raddle/raddle-template-agent.md:49-54`)
  - Embed and registry: `contracts/stencils/stencils.go:214-219,745,894`. Covered: the file path and name are unchanged.
  - Seeding to hubs goes through `internal/stencilstore` (`doc.go:4,16`). Covered: a hash-stamped banner lets an unedited seed refresh, and no prior-hash list needs an entry.
  - The rubric reads this stencil by name (`loom-rubric-raddle-review.md:15`). Covered.
  - `rubric_test.go:53` asserts only that the rubric names `raddle-template-agent`, which still holds. Covered.
  - Proposal spec `contracts/specs/raddle-proposal-spec.md`: it describes the format and checks (`packages-shared` at :97, `order_departures` at :29) and carries no split-judgement prose to update. Covered.
  - Docs: `docs/overview.md:836,1014,1391`, `docs/reference/weaving-glossary.md:155` and `internal/raddleparser/doc.go` describe raddle at a level the rule does not touch. Covered.
  - `PATTERN-stencil-ownership`: still read via `stencilstore`, no engine text, the parent and edit directive markers kept, and no restated format. Covered.
- **Rubric "Judge the split" gains two judgements** (`loom-rubric-raddle-review.md:34-41`)
  - Rubric consumers: the Bouncer seed and judge stencils and the burler review prompt, through ReadRubric (comment at :1-6). Covered: no new marker, and the marker allowlist test in `rubric_test.go` still passes on the existing markers.
  - "Already enforced" (:19-22) tells the reviewer never to re-derive gate checks. The different-packages half the rubric omits is gate check `packages-shared` (`internal/raddleparser/check.go:35,398`; spec :97). Covered, and consistent with the summary's design note.
- **Rubric `design` class sentence rewritten** (:66-68)
  - Rubric consumers read the class text verbatim, and no test or doc asserts it. Covered. Every earlier case is kept: a decision no part can build, a seam that does not hold, and a recipe or profile that does not fit.
- **Commit message** (701ac0d0): describes both files and the three-part rule, with no `Pattern-Departure:` trailer, and none is needed. Covered.
- **Declared PATTERN changes**: the directive declares none. Nothing to land.

**Findings**

1. **NIT, consistency.** Location: `contracts/stencils/raddle/raddle-template-agent.md:48,51`. The new bullet's "parts with no dependency between them touching different packages" restates the bullet directly above it, "keep two parts with no order between them off the same package", so the stencil states the same constraint twice. The board entry asks for the rule's wording, so this is a duplication, not a scope error. Evidence: `raddle-template-agent.md:48` and `:51`.

No other member of the affected set is left wrong.

**Tool calls**

1. grep for the changed rule phrases across the repo, the spec's chain/concurrency text and the `docs/` listing: to find docs and tests that restate the split rules, which the manifest lacked.
2. grep of `docs/reference`, `docs/overview.md`, the spec and the `raddleparser`/`loomengine` docs for raddle, and of the spec for order and package lines: to find raddle docs that may describe the split rules.
3. grep of the raddle passages in `docs/overview.md` and the glossary: to check whether either describes split judgements.
4. grep of `internal/raddleparser/doc.go` and the `raddle` entries in `stencils.go`: to check the parser doc and the stencil registry.
5. grep of `internal/stencilstore` for a prior-hash list: to check whether changing a shipped stencil needs a recorded old hash.
6. grep of `internal/stencilstore/doc.go` for hash handling: to confirm how an unedited seed refreshes.
