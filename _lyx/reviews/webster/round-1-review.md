---
verdict: APPROVED
---

Review range: no `_lyx/loom/rework` round exists, so the range is `git diff $(git merge-base main HEAD)..HEAD` (`product.parent` is `main`).
`lyx gate test` over `internal/planindex`, `internal/planglyph`, `internal/websterengine` and `contracts/stencils` passes.

No findings.
Each focus question and its verdict:

- **Byte-identity.**
  `webster-fragment-card-read-plain.md` and `webster-fragment-fresh-read-plain.md` carry the old body text verbatim, and `1. {{.card_read}}` keeps the continuation indentation.
  `prePackStepOne`, `prePackFreshRead` and `writePrePackBody` in `template_test.go` pin the nil-pack fork prompt and the recovery prompt to those bytes.
  A glyph opt-out language returns nil from `buildKickstartPack`, so it renders as a nil pack; the existing `language-none` row in `beginbatch_test.go` now asserts the plain step 1.
- **Per-card grouping.**
  `kickstartPack` collects distinct member refs in first-seen order and resolves them in one call, and a test counts the calls.
  Each card gets its own `planindex.CardPack`, rendered under `cardPointerPath`, which the card-list bullet shares.
  The empty and unresolved cases go through `webster-fragment-pack-empty` and `webster-fragment-pack-unresolved`, and the tests cover the empty, none, unresolved and handle-only combinations.
- **Card 1.**
  `appendPackAnswer` cuts a `var` or `const` signature at its first newline and trims trailing space, tab and CR.
  Func and method signatures stay verbatim, and the `TestKickstartPack` row covers ungrouped and grouped var and const plus a multi-line func and method.
- **Stencil registration.**
  The four new fragments plus `pack-card` and `pack-unresolved` are embedded in `stencils.go`, listed in `.gitattributes`, and covered in `markers_test.go` and `claims_test.go`.
- **Test economy.**
  The new cases are rows or subtests in existing tests.
  The `kickstartPack` one-call test is package-internal, but only the internal resolve seam can count the calls.
- **Docs.**
  `websterengine/doc.go` and `planglyph/doc.go` describe the per-card pack, the first-line rule and the pack-on reads, and no source is inlined.
