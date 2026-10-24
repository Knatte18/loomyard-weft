# Cut PATTERN.md from 100 entries to 30

`PATTERN.md` is inlined into every spawned agent's prompt, and it had grown past its headroom line (15.6 KB against the 15.5 KB line).
This change cuts it to 30 entries and 6966 bytes without dropping a rule's substance: related entries merge, package-local rules move into their package's doc, and four rules another enforced home already states leave PATTERN.

## What changed

- **Merges.** Twenty-two survivor entries absorb related ones, following the board entry's review list.
  Each survivor's overview line now names every absorbed clause tersely, and its `pattern/` background file gains one section per absorbed entry, carrying that entry's background text with headings demoted a level.
  For example, `PATTERN-gate-slot` now holds self-check parity, the verified tree, the trimpath-safe root and the gate carry; `PATTERN-pane-text-injection` holds the guarded interrupt, the pause hold, the orch pane and the creel store.
- **Moves.** Twenty-one package-local rules leave PATTERN for their owning package's doc: `agentname`, `batcher`, `burlerengine`, `configreg`, `fabricengine`, `friction`, `hubreconcile`, `landingshed`, `loomcli`, `loomshed`, `lyxcwd`, `parentlock`, `planparser`, `reedengine`, `shedengine`, `shedverbs`, `treadleengine` and `tools/sandbox`.
  Where the package doc already stated the rule, only the missing clauses were added.
- **Drops.** `quarry-cgo`, `markdown-link-integrity`, `comment-line-breaks` and `documentation-lifecycle` leave with their background files: `CLAUDE.md`, `docslink_test` and `lyx loom lint-comments` already hold them.
- **Citation sweep.** Every live citation of a removed ID — test failure messages, doc comments, `contracts/specs`, stencil tests, `crucible/`, `docs/overview.md` and the benchmark guides — now names the survivor ID or the owning package doc.
  `cmd/testtiming`'s redundancy report header and its generated report point at `PATTERN-test-speed`'s test economy section.
  Dated reports under `docs/research/` and `testdata/` corpus files keep their text.

No test, scan or enforcement was removed; only the IDs their messages cite changed.

## Design decisions

- `hub-store-gc` joins `PATTERN-fabric-git` rather than `gitrepo-client-boundary` as the review proposed, because it binds `fabricengine/housekeeping.go`.
  The three entries added after the review join the survivor whose subject they restate: `row-entry-gate` into `shed-recipe-registry`, `gate-carry` into `gate-slot`, `tier-tags` into `test-speed`.
- The glyph conversion rule moves into `planparser`'s package doc, not `planglyph`'s as the decision record names: `planparser` already carried the rule's "Glyph Conversion Chokepoint Invariant" section and owns every sanctioned conversion it lists.
- `agentname` and `lyxcwd` keep their package doc in their main file, so the moved section extends that doc rather than creating a `doc.go`.
  `configreg` had no package doc and `tools/sandbox` no command doc, so each gains a `doc.go`.
- `fabricengine`'s doc already stated the mutation record rule in full, so the move only removes its PATTERN pointer.

## Risks

- A moved rule no longer reaches every agent's prompt; an agent editing outside the owning package meets it only through the enforcing test's message or review.
- A merged overview line is terse; a clause that only the background section spells out can be missed by an agent that does not open the file.
