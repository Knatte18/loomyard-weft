# Cut PATTERN.md from ~100 entries to 30 without dropping a rule

`PATTERN.md` is inlined in every spawned agent's prompt, and it had grown past its headroom line (15630 bytes, with 16384 as the cap).
It is now 6966 bytes and holds 30 entries.
No rule is deleted unless another enforced home already states it.

## What changes

- **Merged entries.** Twenty-two survivors absorb their related entries, such as `PATTERN-config-layers`, `PATTERN-fabric-git`, `PATTERN-gate-slot`, `PATTERN-shed-driver`, `PATTERN-pane-text-injection`, `PATTERN-test-speed` and `PATTERN-test-isolation`.
  Each survivor's overview line names every clause tersely.
  Each absorbed entry's background file becomes one `##` section of the survivor's background file, headings demoted one level.
  An absorbed entry with no background file contributes its overview rule as a bullet.
- **Moved entries.** Twenty-one package-local rules move out of the overview into a section of the owning package's `doc.go`, condensed to the rule, its bounds and its enforcement.
  Owners: `configreg`, `hubreconcile`, `lyxcwd`, `fabricengine`, `parentlock`, `shedengine`, `landingshed`, `shedverbs`, `loomcli`, `treadleengine`, `loomshed`, `planglyph`, `batcher`, `burlerengine`, `agentname`, `friction`, `reedengine` and `tools/sandbox`.
  Their background files go with them.
- **Dropped entries.** `quarry-cgo`, `markdown-link-integrity`, `comment-line-breaks` and `documentation-lifecycle` leave PATTERN with their background files.
  `CLAUDE.md` states the cgo build and the documentation lifecycle, `docslink_test` fails clearly on a broken docs link, and `lyx loom lint-comments` runs at every card gate.
- **Citation sweep.** Go code and comments, `contracts/`, `crucible/` and `docs/` now cite the survivor's ID for an absorbed entry, the owning package doc for a moved one, and the enforcing test or `CLAUDE.md` for a dropped one.
  Several test failure messages changed with it.
  Reports under `docs/research/` and `docs/benchmarks/` and `testdata/` corpus files keep their text, since they record past runs.

No test or scan is removed, and `internal/pattern`'s checker and caps are untouched.

## Design decisions

- `hub-store-gc` joins `fabric-git` rather than `gitrepo-client-boundary`, because it binds `fabricengine/housekeeping.go`, not `internal/gitrepo`.
- `row-entry-gate`, `gate-carry` and `tier-tags` were added after the board entry's review; they join `shed-recipe-registry`, `gate-slot` and `test-speed`.
- `shed-recipe-registry` had no background file of its own, so the absorbed `recipe-setting-lookup` file is its base.
- `lyxdirs-single-declarer` keeps its `tools/` carve-out inside `durable-vs-ephemeral-state`.

## Risks to weigh

- A moved rule is no longer in every agent's prompt, so an agent outside the owning package can break a rule that only review catches.
- A merged overview line is terse; a clause that only the background file spells out may be missed.

## To check by hand

- No test checks that a cited `PATTERN-` ID exists. Run `grep -rho 'PATTERN-[a-z-]*' --include='*' . ` outside `docs/research/`, `docs/benchmarks/` and `testdata/`, and confirm each ID is one of the 30 in `PATTERN.md`.
- Skim the new `doc.go` sections for rules whose enforcement you want to confirm survived the condensing.
