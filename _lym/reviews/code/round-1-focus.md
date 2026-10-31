---
round: 1
focus:
  - "internal/summaryparser (LandingMessage and its callers): the credit block is passed where the trailer block used to be; settle whether the landing commit message still gets a blank line before the block, and whether a block with no heading or an empty block leaves the message ending in the body, per D2."
  - "internal/landingshed/prreview.go: the credit block is composed once before the Create/Edit branches; settle whether every Create and Edit path sets the body with the block (D4), and whether a read failure of the engines-run list is an error and not silently dropped."
  - "internal/landingshed/config.go MigrateConfig and RetiredKeys: settle that co_authored_by, trailer_text and engine_trailers load and are pruned with no value carried into another key (D5), and that the remaining renames still behave as before, including the unchanged-on-second-pass guarantee."
  - "internal/landingshed/config.go LoadConfig and ConfigSpec: settle that credit_engines keeps the engine-name check, that all four new keys are cross-pair and credit_engines is still an open map, and that the template loads with credit true and the three texts (D1, D6)."
  - "Leftover references: grep the repo for Lym-Task, TrailerBlock, TrailerText, EngineTrailers, engine_trailers, trailer_text, co_authored_by and Co-Authored-By outside retired-key lists; settle that docs (doc.go, docs/overview.md, the final-summary spec, the describe stencil) and the summaryparser finding detail no longer describe Go appending a trailer."
  - "Test coverage against the D2 and D3 and D5 acceptance criteria: settle that each is covered by a row in an existing table where one exists, that no new test duplicates an existing one, and that no behavior the diff adds (credit off, empty heading, empty text, duplicate engine names, PR body) is left uncovered."
---

Seed focus drawn from the decision record's acceptance criteria and the committed diff of `9daa8ce4`.
