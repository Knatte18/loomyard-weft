```yaml
lens: code-pattern-fit
outcome: reported
```

**Task: code-pattern-fit lens. Each repo rule checked against the changed files, held or broken.**

Rule check:
- **`PATTERN-config-layers`, held.**
  - `runprofile.Load` still reads through `configengine.StackLayers` with no template fill and falls back to the template only when no layer exists (`runprofile.go:105-115`).
  - That is the "bar `runprofile`" stance, and `pattern/PATTERN-config-layers.md:55` still describes it correctly.
  - No PATTERN text needed to change, and the diff changes none.
- **`PATTERN-refusal-way-forward`, held.**
  - Every new refusal names a way forward:
    - `Settings`, for a profile with no block for the recipe (`runprofile.go:90`).
    - The shape errors wrapped in `decode` (`runprofile.go:137`).
    - The `defaults` errors (`runprofile.go:144`).
  - Seed-Child routes the refusal to Stuck through `ErrInvalidRunSettings` (`battencli/runsettings.go:35-38`).
  - I did not read whether a test reaches each refusal; that belongs to the tests lens.
- **`PATTERN-tolerant-records`, held.** Profiles are config, not a Go-written record, and the diff touches no record writer.
- **The one `configreg.PrepareRunSettings` step, held.** `checkProfiles` calls `PrepareRunSettings` once per recipe block (`configreg/runsettings.go:152-156`), and `LayerRunSettings` is still the layering step.
- **Declared PATTERN changes, held.** No PATTERN change is declared in the rubric or the directive, and the diff touches neither `PATTERN.md` nor `pattern/`.
- **CLAUDE.md "Docs land in the same commit", held.**
  - The diff updates all of these:
    - `internal/runprofile/doc.go`
    - `internal/battenshed/doc.go`
    - `internal/boardengine/doc.go`
    - `internal/configcli/doc.go`
    - `internal/boardcli/doc.go`
    - `docs/overview.md`
    - `plugins/ly/skills/board/SKILL.md`
    - the orch stencil
  - This matches the decision record's docs list and adds the SKILL and stencil.
  - The old shape is gone from them.
- **Semantic line breaks in `.md` files, held.** `docs/overview.md`, `SKILL.md` and `orch-template-role.md` keep one sentence per line, and table cells are untouched.
- **Comment conventions, held.**
  - New doc comments open with the symbol name (`Profile`, `Profiles`, `Settings`, `Default`, `decodeProfile`, `decodeDefaults`).
  - File headers keep the `file.go — …` form.
  - No end-of-line comments were added.
  - Line breaks are not checked, by the rubric.
- **Commit-message claims, held.**
  - The message says the template seeds the ladder with hard cap = rounds + 1, a darn-only `docs`, and defaults `loom: solid` and `darn: light`. `template.yaml` matches on every row. `docs` is a darn-only block with `darn.writer: sonnet[medium]`, `code_max_bounces: 1` and `code_hard_max_bounces: 2`.
  - The message does not claim the hub's own profiles were rewritten. The focus directive's wording on that point is not in the commit. The decision record puts the rewrite out of scope, and that is held.
  - No `Pattern-Departure:` trailer is present or needed.
- **Retired keys, held.**
  - My grep for `merge_max_bounces`, `darn.review_` and `darn-wide` found hits only in migration tests that deliberately feed retired keys:
    - `configcli_integration_test.go`
    - `configreg/runsettings_test.go`
    - `configsync_test.go`
  - Those tests use the new profile → recipe → module shape, so they are not stale references.
  - The grep also hit `reviewbudget_test.go:231`, which is `merge_max_bounces` used as a rejected-key example. That test is unrelated to profiles.
  - The grep found no `darn-wide` or `budget` profile in the template, docs, `battencli`, `boardcli`, `SKILL.md` or the orch stencil. The `budget` hits (`loomshed/loompreflight_test.go`, `configengine/layers_test.go`, `configsync`) are a batcher profile key and generic engine fixtures, unrelated to run profiles.

Findings: none.

Tool calls:
1. `grep` over the repo for retired keys and the removed profile names, plus `profile` in `PATTERN.md` and `pattern/*.md`. My context lacked a repo-wide search for leftovers outside the diff.
