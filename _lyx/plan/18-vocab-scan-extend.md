# Card 18 — vocab-scan-extend

**Edit:**
- `internal/lyxcwd/enforcement_test.go`
- `internal/lyxcwd/anchor_test.go`
- `internal/lyxcwd/geometry_test.go`
- `internal/lyxcwd/docslink_test.go`
- `PATTERN.md`
- `pattern/PATTERN-fabric-vocabulary.md`

**Create:**
- `internal/lyxcwd/vocabscan_test.go`

**Intent:** Extend `TestEnforcement_FabricVocabulary`'s bare `weft`/`warp` rule beyond production Go (D10), with the identifier exemption approved by the operator through the orchestrator on 2026-10-04.

Scope added to the bare-token rule:
- every `*_test.go` file outside the owner set, anywhere in the module;
- every `.md`, `.yaml` and `.go` file under `contracts/`, `docs/`, `plugins/`, `crucible/` and `cmd/`;
- `CLAUDE.md` and `README.md`.

Skipped:
- `_lyx`, `.lyx` and `sandbox/` fixture directories;
- a closed path allowlist, each entry with a one-line reason in the test:
  - fabric's own docs (only a doc whose subject is fabric's two-sided design);
  - the measurement reports under `docs/benchmarks/` and `docs/research/`;
  - this scan's own test files, which must spell the tokens they ban.

The `vocab-exempt` table (Shared Decisions) becomes a closed list in the test: owner-set exported identifiers and env-var names a non-owner file may spell.
The scan strips exactly those whole tokens before matching, so any other identifier carrying a side, including a new owner-set export, still fails.
Adding an entry needs a reason line naming its owner-set declaration.
Entries reported as deviations by cards 14–17 are added here only when they are owner-set spellings.

The `weftname` import rule keeps its `internal/lyxcwd/geometry_test.go` exemption.
`anchor_test.go`, `geometry_test.go` and `docslink_test.go` are reworded under the `vocab-sweep` rule, except for `vocab-exempt` identifiers and the `weftname` import.
Failure messages name the offending file and token and stay vocabulary-neutral where they leave fabric (they may quote the token found).

Refactor the scan into a helper over a told root, so `vocabscan_test.go` can run it on `t.TempDir()` fixtures.
The fixture cases:
- fails a `docs/x.md` naming `weft`;
- fails an `internal/foo/foo_test.go` outside the owner set naming `weft`;
- passes the same content at an allowlisted path;
- passes a file spelling only a `vocab-exempt` identifier;
- fails a file spelling a non-exempt identifier such as `FooWeftBar`.

`TestEnforcement_FabricVocabulary` itself passes on the swept tree.
Any straggler cards 14–17 missed is reworded in this card under the same rule.

`PATTERN-fabric-vocabulary`'s line and background file state the widened scope, the allowlist and the exempt-identifier list as closed lists in the test.
The scan's test comment may name the board note `fabric-api-vocabulary` for the out-of-scope rename; the PATTERN text does not.

**ImpactSummary:** Widens one enforcement scan across test files and doc roots; only lyxcwd test files and the PATTERN entry change.
