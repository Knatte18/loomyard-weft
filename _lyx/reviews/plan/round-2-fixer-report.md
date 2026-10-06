# Round 2 fixer report

## Changed

- **F1** — `09-websterengine-untag.md`: the Intent now says webster's integration tests build a scratch git repo with `gitkit.CommitFile` (a seeded worktree), not a `hubforge` hub, and that `PATTERN-hubforge-fixtures` does not apply to them.
- **F2** — `00-overview.md`: card 17 joins the `Applies to:` list of `skill-loading-tests-untouched`.
  `17-measure-after.md` gains the keep-with-`skills-in-one-turn` rule for protected tests and the stub-helper rule for `writeStubDriverScript` and `integrationWriteStubDriverScript`.
- **F3** — `00-overview.md`:
  - `untag-procedure` declares an untag card's write scope as its Targets plus the `internal/testkit/` kit directory, new or existing.
    It says the card's commit message names every kit path it created or changed, and that a `shuttlefake` extension leaves the `SkillLoader` methods alone.
  - `prune-procedure` no longer exempts every prune card from the no-other-package rule; a prune card writes a kit file only when its own Targets name the kit, which today is card 8's `shuttlefake`.
  - The `ImpactSummary` of cards 9, 11, 13 and 15 names the shared-fake write.

## Deferred

- **F3, naming kit files in Targets.**
  The review offered naming the kit's files in each untag card's Targets as one fix branch; it was not taken.
  Which fakes are shared across the six packages cannot be known before the tests are read, so the files cannot be named at plan time.
  The declared write scope covers the gap, and a prune card can no longer write one by accident.

`approved:` in `00-overview.md` was not touched.
