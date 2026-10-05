# Support log — test-suite-measure

## Interview

Autonomous mode: no operator answered; every pick below is an auto-pick recorded in the Question ledger.

1. Read the board task, its dependents (`test-suite-prune-core`, `test-suite-prune-rest`), its dependency (`test-writing-rules`) and note `test-tiers`.
   The prune tasks need a tool, a per-package candidate report and consolidated rules; `test-tiers` keeps gate selection.
2. Explored `cmd/testtiming`, `internal/testkit/tmuxkit`, the build tags in use (`integration`, `smoke`), the smoke files that look up `claude`, `/tmp/tmux-1000/`, the Testing PATTERN entries and backgrounds, and the verify text in the plan spec and template.
3. Found `tmuxkit.Main` already isolates `TMUX_TMPDIR` (landed 2026-10-03); most leaked sockets predate it, but 2026-10-05 runs still left `lyx-001-*`, `lyx-contract-*` and `lyx-Test*` files.
   Settled on a survey-then-fix design with one shared cleanup helper and a bounded production removal (D9).
4. Tier names: kept `integration` for tier 2, named tier 3 `tmux` and tier 4 `llm` (D6); integration files that start tmux move to `tmux`.
5. Noticed retagging changes the integration tier's wall time by itself, so the speed criterion is measured on the pre-retag test set (D4).
6. Plan verify text: `llm` is compile-only; `tmux` is token-free and may be run; gate choice stays in `test-tiers` (D10).
7. Redundancy: coverage blocks per top-level test, subset of the union of others, with covering tests named and a greedy removable set (D2).
8. LLM guard through a single `llmkit` helper and a static guard test (D7).
9. Accepted the board's consolidation proposal (D8).

## Rejected alternatives

- New `cmd/testprune` beside `cmd/testtiming`: duplicates `go test -json` parsing and splits suite measurement across two tools.
- Redundancy against a single other test only: misses tests covered by a combination of others, which is the common case for table tests.
- Redundancy without a removable set: two mutually covering tests would both be listed and could both be deleted.
- Package-local `-coverpkg`: a CLI test's coverage of the engine it drives would not count, so CLI tests would look uniquely covering when they are not.
- Renaming `integration` to `git`: churns about 250 files, `cardGateCommand`, every plan verify and verified-tree records for a name change the operator did not ask for.
- Naming tier 3 `live` or `e2e`: the operator's objection to `smoke` was that it says nothing about tmux.
- Leaving tmux-starting integration tests in `integration`: contradicts the operator's tier split by need.
- Running the `tmux` tier in every gate now: gate selection belongs to note `test-tiers`.
- A runtime LLM guard (stripping `claude` from `PATH` in `tmuxkit.Main`, or an env flag the provider reads): fragile, and the env flag would put a test hook into production code.
- Rewriting LLM-spawning tests onto fakes: the board asks for a retag; rewriting is the prune tasks' kind of work.
- Cleaning the existing 600-odd socket files: this task stops new leaks; old files are the operator's to delete.
- Keeping the three Testing entries separate and only adding the tier rules: grows the section, which the operator rejected.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick): Where does the measurement tool live? — Extend `cmd/testtiming` (D1).
- Q2 (auto-pick): What counts as redundant? — Covered blocks ⊆ union of the package's other tests, top-level tests, module-wide `-coverpkg`, greedy removable set, "no coverage" listed apart (D2).
- Q3 (auto-pick): Where are the reports committed? — `docs/benchmarks/test-suite-timing.md` and `docs/benchmarks/test-redundancy.md` (D3).
- Q4 (auto-pick): How is "dropped from the speed fixes alone" measured when retagging moves tests? — On the pre-retag integration set, before the retag (D4).
- Q5 (auto-pick): Tier names? — untagged, `integration`, `tmux`, `llm` (D6).
- Q6 (auto-pick): Do integration files that start tmux move? — Yes, to `tmux` (D6); the card-gate impact is an open risk for note `test-tiers`.
- Q7 (auto-pick): How is "no tier below llm spawns an LLM" enforced? — `llmkit` helper plus a static guard test, review for the rest (D7).
- Q8 (auto-pick): Consolidation shape? — The board's proposal: five entries (D8).
- Q9 (auto-pick): Does production reed get a socket fix? — Only if the survey shows its teardown leaves a socket file, with the removal bounded to its own killed key (D9).
- Q10 (auto-pick): What does the plan template say about `tmux`? — Token-free, may be run by a plan verify; `llm` compile-only (D10).
