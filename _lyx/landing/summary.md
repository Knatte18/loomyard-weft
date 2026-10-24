# Cut PATTERN.md from about 100 entries to 30

`PATTERN.md` is inlined in every spawned agent's prompt, so its size is paid on every run.
It had grown past its headroom line and needed 20 entries shortened to fit the last addition.
It now holds 30 entries in about 7 KB, down from about 15.6 KB, with no rule dropped except the four listed below.

## What changes

- Related entries merge into 22 survivors, for example `config-layers`, `fabric-git`, `gate-slot`, `shed-driver`, `pane-text-injection` and `test-speed`.
  A survivor's overview line states the whole rule tersely.
  Its `pattern/` background file keeps its own text and gains one `##` section per absorbed entry, headings demoted one level.
  An absorbed entry with no background file contributes its rule as a bullet.
- 21 package-local rules move into the owning package's `doc.go`, condensed to the rule, its bounds and its enforcement.
  The packages are `configreg`, `hubreconcile`, `lyxcwd`, `fabricengine`, `parentlock`, `shedengine`, `landingshed`, `shedverbs`, `loomcli`, `treadleengine`, `loomshed`, `planglyph`, `batcher`, `burlerengine`, `agentname`, `friction`, `reedengine` and `tools/sandbox`.
- `quarry-cgo`, `markdown-link-integrity`, `comment-line-breaks` and `documentation-lifecycle` are dropped with their background files.
  `CLAUDE.md` states the cgo build and the documentation lifecycle, `docslink_test` fails with a clear message on a broken docs link, and `lyx loom lint-comments` runs at every card gate.
- Every live citation of a removed `PATTERN-` ID is swept across Go code and comments, `contracts/`, `crucible/` and `docs/`.
  An absorbed ID becomes its survivor's ID, a moved ID names the owning package's doc, and a dropped ID names the enforcing test or `CLAUDE.md`, or is removed.
  Test messages that cite an ID change with it; no test or scan is removed.

## Design decisions

- `hub-store-gc` joins `fabric-git`, not `gitrepo-client-boundary`: it binds `fabricengine/housekeeping.go`, not `internal/gitrepo`.
- `row-entry-gate`, `gate-carry` and `tier-tags` were added after the task's review list and join `shed-recipe-registry`, `gate-slot` and `test-speed`.
- `shed-recipe-registry` had no background file of its own and takes the absorbed `recipe-setting-lookup` file as its base.
- Reports under `docs/research/` and `docs/benchmarks/` and corpus files under `testdata/` keep their old `PATTERN-` citations, as records of a past run.
- `internal/pattern`'s checker and caps are untouched.

## Risks

- A moved rule is no longer in every spawned agent's prompt, so an agent outside the owning package can break a review-only rule without seeing it.
- A merged overview line names several clauses tersely; an agent may miss a clause that only the background file spells out.

## To check

- No test checks that a cited `PATTERN-` ID exists.
  Run `grep -rn 'PATTERN-' --include='*.go' --include='*.md' --include='*.yaml' .` outside `docs/research/`, `docs/benchmarks/` and `testdata/`, and confirm each ID is in `PATTERN.md`.
