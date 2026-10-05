# Webster audit recovery: all 15 batches done

Every batch reported `status: done` through `record-batch`.
Batch 12 needed one re-fork: its first fork died on an API error before writing a report, and the second fork kept the uncommitted work and finished it.

## Shuttle: hung background shells (cards 1–4)

- 01 (`2074f1c5`): `claudeengine` reports a waiting turn end's outstanding tasks as a neutral fork/shell list (`BackgroundTask`, `Event.Outstanding`).
- 02 (`2324a543`): `background_shell_wait_min` joins `shuttle.yaml`, default 10, non-positive refused.
- 03 (`0bf70d5d`): the wait loop counts a turn end once non-awaited shells outlast the bound; `Spec.AwaitedShellPrefixes` and `Result.ExpiredShells` added.
  Deviation: `attach_test.go`, `startup_test.go` and `completionsignal_enforcement_test.go` changed, because `Result` became non-comparable.
- 04 (`57468159`): a `PreToolUse(Bash)` hook prefixes each command with `exec </dev/null; ` and grants no permission.
  Deviation: `internal/shuttlecli/modelpin_test.go`, for the spawn-site count.
  Unverified: the fork did not check whether `updatedInput` replaces the whole `tool_input`, which would drop fields such as `run_in_background`.
  The card's smoke test (not run, needs a live `claude`) would not catch it.

## Webster: expired shells and contract files (cards 5–7, 10, 13)

- 05 (`a852e24d`): Master awaits `recover-batch`; a waited-out shell becomes a warning, a `## Background shells waited out` summary section and a friction note.
- 06 (`15f36a77`): fork and fixer prompts name the outcome and summary paths as files never to write.
- 07 (`4ba69c5e`): audit facts carry each write's time and result (`WriteEvent`).
- 10 (`0f50dd2e`): `loadRunWrites` is the single reader of every Master and fork write in the run's sessions.
- 13 (`1d8f1e62`): a fork-contract-write finding clears on file absence or Master's later successful write; the refusal-spec rows are updated.

## Reset verb and way-forward text (cards 8, 9, 11, 12, 14, 15)

- 08 (`b6d777c9`): `Fabric.ResetPairWarp`, a gated primitive that exempts the run's own dirt.
- 09 (`2c999b29`): the verify gate persists and clears `State.PreFixHead`.
- 11 (`fd49e0ef`): `PlanReset` resolves `start` and `pre-fix` targets and refuses unsafe resets before fabric is called.
  Deviation: `gitwrap.go` and `doc.go`.
- 12 (`3e3af182`): `lyx webster reset --to start|pre-fix` with its refusal-spec rows.
  Deviation: `internal/lyxcwd/enforcement_test.go` (`vocabExempt` entry).
- 14 (`305a5a2b`): audit and fresh refusals name each finding once with reasons and a numbered command list; `RunDeps.ReentryStep` gives a single re-entry step.
  Deviation: `contractevidence_test.go`, `internal/websterengine/doc.go`, `internal/shedadapters/doc.go`.
- 15 (`48a6decd`): the remaining reset texts name the verb; `PATTERN-no-denied-recovery` and its scan in `cmd/lyx/norecoverydeny_test.go` landed.
  Deviation: `internal/websterengine/suspect_test.go`.

## Open for the operator

- The plan's departure from D7 stands: the report-head and last-batch-head refusals name plain `git reset --keep <sha>`, not `lyx webster reset`.
  The operator confirms it or widens the verb's targets.
- `record-batch` logged stencil-drift warnings (board copy behind the worktree source) for the master, verify-fix and fork-prefix stencils.
  A production deploy via `update-plugins.sh` is the operator's call; none was run.
