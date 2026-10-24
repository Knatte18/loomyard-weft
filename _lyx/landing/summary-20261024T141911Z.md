# Cut PATTERN.md from about 100 entries to 30

`PATTERN.md` is inlined in every spawned agent's prompt, so its size is paid on every run.
It had grown past its headroom line (15630 bytes), and a recent change only fit by shortening 20 entries.
It now holds 30 entries in about 7 KB, with no rule dropped except four that another enforced home already states.

## What changes

- **Merged:** 22 survivor entries absorb the related entries that restated one facet of the same subject, for example `config-layers`, `fabric-git`, `gate-slot`, `shed-driver` and `pane-text-injection`.
  A survivor's `pattern/` background file keeps its own text and gains one section per absorbed entry, so no rule's detail is lost.
- **Moved:** 21 package-local entries become a section in their owning package's `doc.go`, condensed to the rule, its bounds and its enforcement.
  The owners are `configreg`, `hubreconcile`, `lyxcwd`, `fabricengine`, `parentlock`, `shedengine`, `landingshed`, `shedverbs`, `loomcli`, `treadleengine`, `loomshed`, `planglyph`, `batcher`, `burlerengine`, `agentname`, `friction`, `reedengine` and `tools/sandbox`.
- **Dropped:** `quarry-cgo`, `markdown-link-integrity`, `comment-line-breaks` and `documentation-lifecycle`, with their background files.
  `CLAUDE.md` states the cgo build and the documentation lifecycle, `docslink_test` fails clearly on a broken docs link, and `lyx loom lint-comments` runs at every card gate.
- **Citations swept:** every live citation of a removed `PATTERN-` ID in Go code and comments, `contracts/`, `crucible/` and `docs/` now names the survivor's ID, the owning package doc, the enforcing test, or nothing.
- No test or scan is removed, and `internal/pattern`'s checker and caps are untouched.

## Design decisions

- `hub-store-gc` joins `fabric-git`, not `gitrepo-client-boundary`: it binds `fabricengine/housekeeping.go`, not `internal/gitrepo`.
- Three entries added after the board's review are folded in: `row-entry-gate` into `shed-recipe-registry`, `gate-carry` into `gate-slot`, `tier-tags` into `test-speed`.
- `shed-recipe-registry` had no background file, so the absorbed `recipe-setting-lookup` file is its base.
- Citations in measurement reports under `docs/research/` and `docs/benchmarks/` that record a past run, and in `testdata/` corpus files, keep their text.
- Merged overview lines are terse and name several clauses.
  An agent may miss a clause only the background file spells out, and an agent outside a moved rule's package no longer sees that rule in its prompt.

## To check by hand

- No test checks that a cited `PATTERN-` ID exists.
  Run `grep -rn 'PATTERN-' --include='*.go' --include='*.md' .` and compare each ID against `PATTERN.md` and `pattern/`, to catch any stale citation outside the report and `testdata/` exclusions.
- Read the merged background files in `pattern/` for a clause lost or duplicated in a merge.
- `docs/benchmarks/running-tests.md` and `docs/benchmarks/test-redundancy.md` were edited although benchmark reports are meant to keep their text; confirm the edits only fix live links.
