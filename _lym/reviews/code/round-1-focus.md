---
round: 1
focus:
  - "`internal/driftengine/paths.go`, `capture.go`, `steprun.go`: does `{meta_dir}` resolve to a directory outside `{baseline_dir}` for both `diff: lines` and `diff: command`, and is it kept with the capture while never listed, compared or shown as drift?"
  - "`internal/driftengine/expect.go`, `expectation.go`, `records.go`, `contracts/stencils/drift/drift-template-expect.md`: does the expect agent mark metadata files and keys under both diff modes, and does the scope record carry the marks from the seal on?"
  - "`internal/driftengine/check.go`, `diff.go`, `verdict.go`, `verify.go`: does every later listing, expectation, check and gate drop the marked cells, and does a marked file or key never surface as 'not equal' in a prompt, listing or verdict?"
  - "`internal/driftengine/check.go`, `verdict.go`: does the gate end at once, with no re-prompt, only when every unexpected cell is one the writer cannot change; does a mixed failure still re-prompt; and does the parent get one question, not one from the gate and one from the driver?"
  - "`internal/driftengine/check.go` commit-comparison path: how are the two captures' commits read and compared, and what happens when a commit is unreadable, missing or identical across both captures?"
  - "`internal/driftengine/config.go`, `template.yaml`, `doc.go`, `contracts/recipes/darn-recipe.yaml`, `internal/shedrecipe/entries_gate.go`: does the template and `drift init` text say provenance goes to the metadata directory, does no `volatile_files` or `volatile_keys` entry become necessary for metadata, and is the recipe's capture command consistent with the new placeholder?"
  - "Regression check output: does it name the baseline commit and the marks, per the board entry's Wanted section; and does the landing summary claim nothing the diff does not bear out, including the Models hub harness change the entry says the landing must name?"
  - "Scope against the board entry: does the diff make Lym write no `_source.txt` itself, and does it add nothing beyond what the entry licenses?"
  - "Tests in `capture_test.go`, `check_test.go`, `expect_test.go`, `expectation_test.go`, `verdict_test.go`: is each new behavior covered once, with no duplicate of an existing test and no assertion on static prose?"
---

The decision record holds only the board entry under `## Goal`, so the entry is the measuring stick.
Its Wanted section lists the behaviors each focus entry maps to.
