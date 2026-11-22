---
round: 1
focus:
  - "internal/loomcli/hardenwiring.go `hardenLiveArgs`: `build.IsLocalImport` is true only for `.`, `..`, `./x` and `../x`. Settle what the prefix does to an absolute path, a `...` pattern or an empty entry, and whether the decision record's wording ('a relative path that lacks it') licenses that behavior."
  - "internal/loomcli/hardenwiring_test.go `TestHardenLiveArgs`: settle whether the test covers the `./`-already-present case and the bare case against independent literals, and whether any other test or caller of `hardenLiveArgs` or `hardener.packages` (for example `CheckCampaign`) still assumes verbatim packages."
  - "The `Root-Cause:` trailer on d95d266ff: settle whether it names the defect the diff removes, and whether the commit message and doc comment claims match the code."
---

Single small diff: one function and its test. The record's `## Goal` entry is the measuring stick.
