# landing.yaml: operator-set trailer_text and per-engine engine_trailers

The landing commit's trailer block was a single `Co-Authored-By: <value>` line, with the value from `landing.yaml`'s `co_authored_by`.
The key refused an empty value, and the fixed prefix left the operator no say over what the block holds.

`co_authored_by` is replaced by two keys, each copied verbatim with no prefix added:

- `trailer_text`: fixed lines, possibly several, that land on every commit.
- `engine_trailers`: a map from engine name to trailer text.
  An engine's text lands only when that engine ran at least one session in the task.

An empty `trailer_text`, an empty map, or an engine left out or set to empty text adds nothing.
With both empty the commit has no trailer block.
The template default keeps today's landing commit for a Claude-only run: `trailer_text` empty and `engine_trailers.claude` set to `Co-Authored-By: Claude <noreply@anthropic.com>`.
The change description still may not carry trailers of its own, and the Describe validation keeps refusing them.

## Design decisions

- **Composition.** The block is `trailer_text` trimmed, then each ran engine's trimmed text in sorted engine-name order, joined by newlines.
  The landing message is the commit message, a blank line and the block, or the bare commit message when the block is empty.
  `summaryparser`'s `LandingMessage` now takes the composed block instead of a co-author value.
- **Which engines ran.**
  Per-run directories under the shuttle run directory root are swept once their strand ends, so they alone cannot answer at landing.
  `shuttleengine`'s `Start` therefore records each started run's engine name in an engine ledger file at that root, written through `internal/state`.
  The orphan sweep removes directories only, so the file survives.
  `Runner.EnginesRun` returns the sorted union of the ledger and the engine of every run record still present, a record with no engine reading as the default engine.
  `landingshed` reads it through a new `Deps.EnginesRun` seam, wired in `loomcli`; a nil seam names no engine.
- **Ledger semantics.**
  The ledger counts start attempts that passed `Prepare`, so a start that fails after the ledger write can leave an engine counted that never ran a session.
  A failed ledger write fails `Start`, removes the run directory and registers no strand.
  The ledger lives in the never-tracked run directory and dies with the worktree.
  A run in flight across an upgrade, whose earlier run directories were swept before the ledger existed, lands without that engine's line unless a later session of the same engine starts.
- **Validation.**
  An `engine_trailers` key must be an engine `modelspec` knows; an unknown key fails `LoadConfig`, naming the key and the known engines, so a typo cannot silently drop a line.
  Text values, empty included, are unrestricted.
- **Config categories.**
  Both keys are `CategoryCrossPair`, as `co_authored_by` was, and `engine_trailers` is an open map so `lyx config reconcile` carries the operator's entries whole.
- **Migration.**
  `landingshed.MigrateConfig`, landing's `Migrate` hook, rewrites `co_authored_by: X` into `engine_trailers.claude: "Co-Authored-By: X"` and drops the old key.
  An empty, whitespace-only or non-scalar `co_authored_by` is dropped without creating an entry, since empty meant off.
  A document already holding `engine_trailers` keeps it and drops `co_authored_by`, reporting the dropped value.
  A second pass changes nothing.

`internal/landingshed/doc.go`, `internal/shuttleengine/doc.go`, `docs/overview.md`, the final-summary spec and both describe stencils are updated to the new keys.
The `test-budget.yaml` rows for `internal/shuttleengine` and `internal/landingshed` are raised for the new tests.

## To check by hand

Each item below was raised in review and addressed in the diff; none was re-run end to end.

- `internal/shuttleengine/run.go` (`Start`): a failed engine-ledger write fails `Start`, removes the run directory and registers no strand.
- `internal/landingshed/config.go` (`migrateCoAuthoredBy`): an empty, whitespace-only or non-scalar `co_authored_by` migrates to no `engine_trailers.claude` entry rather than a bare `Co-Authored-By:` line.
- `internal/shuttleengine/run.go` (`Start`): when `AddStrand` fails after the ledger write, the ledger keeps the engine, so a landing can carry a line for an engine that never started a session.
- `internal/shuttleengine/enginedispatch_test.go`: the real orphan sweep leaves `engines-run.json` and its lock file in place, and `EnginesRun` still returns the engine.
- `internal/landingshed/finalize.go` (the `EnginesRun` nil guard): a nil seam lands with no engine line.
- `internal/landingshed/config_test.go` (`TestMigrateConfig`): existing `engine_trailers` entries are kept beside `co_authored_by`, and an unparsable or non-mapping document returns `configengine.ErrInvalid`.
- `contracts/stencils/landing/landing-template-describe.md` and `landing-template-describe-darn.md`: both say the landing step appends the configured trailer block, and still forbid a `Co-Authored-By` line in the description.
- `internal/landingshed/deps.go` and `internal/shuttleengine/enginesrun.go`: the `EnginesRun` and ledger doc comments state only what the symbol does and why.
