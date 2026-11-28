---
round: 2
focus:
  - "internal/loomshed/recordseed.go, internal/loomcli/wiring.go and internal/loomcli/sharedbootstrap.go: is the seed parameter that records the parent branch named by one constant at every writer and reader?"
  - "internal/loomshed/recordseed_test.go and the refusals Record-Seed raises (internal/loomshed/recordwrite.go): does a test reach each new refusal and assert its message, and is the cancel path before the stored-only Stuck covered?"
  - "recordSeedDeps in internal/loomcli/wiring.go: does a test pin the recipe, driver, parent, seed path and slug it supplies, with expected values independent of the code under test?"
  - "internal/loomshed/recordwrite.go, the missing-support-log branch in the stored-only mode: does the log written state only what happened, including when no stored record exists?"
  - "contracts/specs/shed-recipe-spec.md, the headers and comments in contracts/recipes, internal/loomshed/loomshed.go and internal/loomshed/doc.go: do they agree with the block-row rule in internal/shedbuild, the loom contract check, and the files that now hold each row?"
  - "_lym/landing/summary.md: does the Merge danger door and blast radius cover what the diff can break (recipe names and row names that another binary reads, batten, shed), and does the text avoid naming the tooling?"
---
