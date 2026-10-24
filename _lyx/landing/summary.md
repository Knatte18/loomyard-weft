# Cut PATTERN.md from ~100 entries to ~30

`PATTERN.md` is inlined in every spawned agent's prompt, so its size is paid on every run.
It had passed its headroom line, and the previous task got through only by shortening 20 entries.
The overview drops from about 15.6 KB to about 7 KB, with no rule removed.

## What changes

- **Merged entries.**
  Twenty-two surviving entries each absorb their related entries, for example `config-layers`, `fabric-git`, `gate-slot`, `shed-driver`, `pane-text-injection`, `test-speed` and `test-isolation`.
  Each absorbed entry's background file becomes one `##` section in its survivor's file, headings demoted one level.
  An absorbed entry with no background file contributes its overview rule as a bullet.
- **Moved entries.**
  Twenty-one package-local rules leave the overview and go into a section of the owning package's `doc.go`, condensed to the rule, its bounds and its enforcement.
  Their background files are deleted.
  Owners include `configreg`, `hubreconcile`, `lyxcwd`, `fabricengine`, `parentlock`, `shedengine`, `landingshed`, `shedverbs`, `loomcli`, `treadleengine`, `loomshed`, `planglyph`, `batcher`, `burlerengine`, `agentname`, `friction`, `reedengine` and `tools/sandbox`.
- **Dropped entries.**
  `quarry-cgo` and `documentation-lifecycle` are stated in `CLAUDE.md`.
  `markdown-link-integrity` is enforced by `docslink_test` with a clear message.
  `comment-line-breaks` is enforced by `lyx loom lint-comments`.
  Their background files are deleted.
- **Citations swept.**
  Go code and comments, test messages, `contracts/`, `crucible/` and `docs/` that cited a removed ID now cite the survivor's ID, the owning package doc, or the enforcing test.
  Reports under `docs/research/` and `docs/benchmarks/`, and `testdata/` corpus files, keep their text because they record a past run.

No checker, test or scan is removed.
The diff to Go test files only changes the PATTERN IDs in messages and comments.
`internal/pattern`, its checker and its caps are untouched.

## Design decisions

- `hub-store-gc` joins `fabric-git`, not `gitrepo-client-boundary`, because it binds `fabricengine/housekeeping.go`, not `internal/gitrepo`.
- The entries added after the original proposal, `row-entry-gate`, `gate-carry` and `tier-tags`, join the survivor whose subject each restates: `shed-recipe-registry`, `gate-slot` and `test-speed`.
- `shed-recipe-registry` had no background file of its own, so it takes the absorbed `recipe-setting-lookup` file as its base.
- A moved rule is no longer in the inlined overview.
  An agent working outside the owning package sees it only through the package doc or the enforcing test's message.
- A merged overview line states several clauses tersely, and a clause may be spelled out only in the survivor's background file.

## To check by hand

- No test checks that a cited `PATTERN-` ID exists.
  Run `grep -rhoE 'PATTERN-[a-z-]+' --include='*.go' --include='*.md' --include='*.yaml' . | sort -u` outside `docs/research/`, `docs/benchmarks/` and `testdata/`, and compare it with the IDs in `PATTERN.md`.
- Read the merged overview lines for a dropped clause against the removed entries in the base commit.
