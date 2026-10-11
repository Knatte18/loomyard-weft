# Card 3 — `lyx loom step` and `lyx loom run` help name both seeding routes

**Edit:**
- `internal/loomcli#loomVerbTexts`
- `internal/loomcli/cli_test.go`
- `cmd/lyx/testdata/test-budget.yaml`
- `docs/overview.md`

**Intent:**
Decision D3.
A bench agent followed `step`'s help, which says step bootstraps "exactly as `lyx loom start` does", to `lyx loom start` and got an LLM driver it did not want; no text on its path named the driverless route.

`loomVerbTexts`'s `Step.Long` drops the claim that step bootstraps exactly as `lyx loom start` does.
It says step needs a seed at the addressed run-id and refuses without one, since it never writes a seed; with one, it runs start's seed-agreement, status-seeding and commit block, then drives exactly one producer.
It names the two ways to seed: `lyx loom start`, which seeds and spawns the LLM driver, and `lyx shed seed self --recipe loom --driver go --param parent=<parent-branch>`, which writes a driverless seed, `<parent-branch>` being the pair's recorded parent branch, `parent_branch` in fabric's origin record.
It says that after a driverless seed the first driving call is `lyx loom step`, whose bootstrap seeds the status file, and that from then on `step` or `run` drives.
The `Short` drops "bootstrap idempotently" in favour of wording that does not read as seeding, for example "drive exactly one producer of a seeded run, reporting a JSON envelope".
`Run.Long`'s paragraph on seeding names the same two routes in the same order: `run` needs the status file, which only `lyx loom start` or a first `lyx loom step` writes, so after a driverless seed `lyx loom step` comes first.
The help names the origin record by its field and never spells a `_lyx` path (`PATTERN-lyxdirs-single-declarer`); it names no denied command (`PATTERN-no-denied-recovery`, which scans `internal/loom*` string literals).

`docs/overview.md`'s batten `step` sentence, which ends "the same single-producer primitive `lyx loom step` is" right after "seeding a fresh run first", is reworded so it no longer reads as if `lyx loom step` seeds a fresh run: batten's `step` seeds when no run is persisted, while `lyx loom step` is the single-producer primitive of the same shape over an already-seeded run.

Coverage: a new table-driven `TestLoomVerbHelp_NamesBothSeedingRoutes` in `cli_test.go` reads each command `shedverbs.Verbs(loomVerbTexts, …)` returns for `step` and `run` and asserts its `Long` holds `lyx loom start`, `lyx shed seed self --recipe loom --driver go --param parent=<parent-branch>`, `parent_branch` and `lyx loom step`, with the driverless seed named before `lyx loom step`; for `step` it also asserts the `Long` no longer holds `exactly as "lyx loom start" does` and says step refuses without a seed.
No existing test asserts this help, so it is a new behavior's test; `internal/loomcli`'s row in `test-budget.yaml` rises by one with it.

**ImpactSummary:** help text of two loom verbs and one overview sentence; no behavior changes.

**Commit:** 3: loom: step and run help name the driverless seed beside loom start
