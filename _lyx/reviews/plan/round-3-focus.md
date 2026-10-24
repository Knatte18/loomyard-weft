---
round: 3
focus:
  - "For each card in _lyx/plan/ that changes a field, config key, signature, rendered string or value type (at least the cards rendering the gate call, re-signing Env/Deps members, changing Targets.Guards, and removing the Command row switch key): search the module's tests, fixtures and docs for the changed name and for the old literal output, and settle whether every hit is listed on that card or an earlier one, including integration-tagged tests, intentionallyNil-style allowlists and doc.go or docs/overview.md paragraphs."
  - "In the retag rules of _lyx/plan/00-overview.md, the retag cards and the D13 follow-up entry: settle whether every file naming an old tag spelling (integration, tmux, llm) is retagged or covered, including non-test support files, TestMain companions constrained by !integration, and testdata packages loaded under a tag, so that none breaks once the hub tier commands drop the old tags."
  - "In card 20's test plan: settle whether each planned new test or assertion pins behavior no listed existing test in battencli already covers."
  - "For each card's Extends group: settle whether any listed ref has existing content rewritten, reordered or removed by the card's Intent, which would belong under Edit."
---

Focus targets the open affected-set method gap and the new retag-support and typing findings.
