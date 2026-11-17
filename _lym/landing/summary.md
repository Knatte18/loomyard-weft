# Launch manifests per engine, per-seat parity test, lym shuttle manifest

Each engine's `Prepare` now reads a launch manifest back from the launch it produced, and a test holds Codex to what Claude delivers.
A Codex delivery that is dropped now fails a test, instead of going unnoticed until a session lacks it.

## What changes

- **Manifest record.**
  `internal/shuttleengine` defines the manifest: one item per thing the session receives, keyed by channel and an engine-neutral key.
  The channels are prompt engine, edit directive, standing passages by heddle slug, instruction sources by origin, skill loads,
  skill listing, plugin session-start contexts, guard rows and spec-level denies, the batching nudge, and notices.
  `claudeengine` and `codexengine` each build it from their own `Prepare`.
  It is written through `internal/state` into the run directory, with each plugin context marked delivered or omitted;
  an omitted item counts as missing in the record's verdict.
  A copy named by the run id goes into a `manifests` directory at the run-directory root and survives a done run's cleanup.
  The orphan sweep and the run lookup by strand skip that directory.
- **Heddle keys.**
  Standing passages and heddle file entries carry their slug and the engine they are keyed to (`internal/heddleengine`).
  An item from a shared target (`agents-file`, `agents-prompt`) is compared.
  An entry under a single-engine target (`claude-file`, `claude-prompt`, `codex-file`, `codex-prompt`) is keyed to that engine and not compared.
  Engine settings are listed and not compared.
- **Parity test.**
  `internal/shuttleparity` builds one launch spec per engine for every seat in `seatengine`'s tables and every other launch-spec producer,
  runs both `Prepare`s on fixtures with a faked sandbox probe and hook runner,
  and fails when the Codex manifest lacks a compared item of the Claude manifest.
  A producer outside `seatengine` joins through a producer table.
  A scan fails on a production `shuttleengine.Spec` literal that sets `Prompt` when its enclosing function has no table entry;
  a literal with no `Prompt` (the attach probes in `internal/shedadapters/bouncer.go`) launches nothing and is skipped.
  `judgeCall` and `runSeedSpawn` had their prompt render split out so the table can call it per engine.
- **`lym shuttle manifest <seat>`.**
  Prints each engine's manifest for the seat in the current worktree, plus the parity verdict, and exits non-zero naming each missing item.
  It shares the sandbox probe and hook runner with the test.
- **Pattern.**
  `PATTERN-engine-parity` is new, and `PATTERN-shuttle-provider-seam` and `docs/overview.md` mention the manifest.

## Design decisions

- Every seat is compared, including those Codex cannot start today (fanned burler review and fix, Plan-Write, Discussion, Merriam, the loom driver).
  `Prepare` does not refuse an unmet requirement, only `Start` does, so the manifest exists for each.
- The `webster-fork-deny` guard item is compared only when the engine meets `RequirementForks`.
  Codex does not, so there it is listed and not compared; a test fails the day Codex reports fork support without delivering the deny.
- `Seat.Specs` clears `Model`, `Effort` and `Version` for each engine.
  They reach only the launch command settings, which are listed and not compared.
- `ReferenceEngine` (claude) and `ComparedEngine` (codex) are named constants in `internal/shuttleparity`:
  the check is between two named engines by definition.
  The commit that adds them carries a `Pattern-Departure` trailer naming `PATTERN-code-conventions`.
- Bounds:
  nothing removes the `manifests` copies;
  `lym shuttle manifest` starts no session, runs no plugin hook or sandbox probe, and lists plugin contexts unrun
  (its verdict compares them by plugin and hook key alone);
  it writes only to its temporary run directory and the Codex home a Codex `Prepare` readies, and keeps no manifest copy;
  its size check of `developer_instructions` leaves out plugin session-start contexts, so a seat it passes can still be refused at a real launch.
- Interim `testOnlyCodeAllowed` entries added for the staged work are removed again when the verb lands.

## To check by hand

- The bench rule (D11) lives on the board, not in this diff.
  Confirm that project `codexengine` states it under its Rules, and that `codex-writer-bench`'s body ends with it.
- Findings the reviews left open, to check in the code:
  - `claude-local-instruction-name-redeclared`: a local instruction name redeclared in `claudeengine`.
  - `codex-drops-omitted-hook-without-plugin`: how `codexengine` handles an omitted hook that names no plugin.
  - `heddle-file-target-engine-literals`: engine name literals in the heddle file-target code.
  - `manifest-fields-unrun-and-verdict-untested`: the unrun fields and the verdict lack test coverage.
  - `manifest-report-of-untested`: the manifest report function lacks test coverage.
  - `parity-engine-names-hardcoded`: engine names hardcoded in `internal/shuttleparity` (covered by the `Pattern-Departure` above).
  - `production-passage-reader-untested`: the production passage reader (`cmd/lym/passagereader.go`) has a test now; confirm it is enough.
  - `report-tempdir-removal-untested-on-errors`: removal of the report's temporary directory on error paths.
  - `interim-reachability-not-admitted-in-allowlist`: the interim allowlist entries and their later removal.
  - `seat-specs-keeps-claude-model-params`: `Seat.Specs` model parameters, answered by clearing them as above.
  - `shuttleparity-package-lacks-testmain`: `internal/shuttleparity/testmain_test.go` is the `TestMain` the package needs.
