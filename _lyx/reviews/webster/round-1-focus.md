---
round: 1
focus:
  - "Master stencil: does the read set name only the plan's `00-overview.md` and the `pattern/` files its constraints section points at, and does every remaining sentence in the template and its docs agree, with no leftover instruction to read cards, sources, tests or docs?"
  - "The three live FRESH-READ fragments and the fork prefix: do they all say no code reaches the fork from Master, and is `webster-fragment-fresh-read-pack-inline.md` byte-identical to its baseline pinned by `inlineFreshReadBody` in `internal/websterengine/template_test.go`?"
  - "Recovery prefix and verify-fix stencil: do they orient from the overview and the rules, and tell the fixer or recovery strand to read the code behind each card or finding itself, without contradicting the Master wording?"
  - "Pins: does every claim row in `contracts/stencils/claims_test.go` and every pin in `internal/websterengine/template_test.go` extend an existing row, assert a short phrase, and match the new wording, with no new top-level test and no stencil marker added or removed?"
  - "Docs: do `doc.go` and any other doc describing the Master's reads match the new behavior, and does the diff change any Go behavior beyond wording?"
---

Wording-only change across four cards; the main risk is a stale sentence left in a stencil or doc that still contradicts the new read set.
