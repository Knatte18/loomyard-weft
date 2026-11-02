# Lens review

Every lens fork returned a report: coverage, row-validity, forced-edits, guard-reach and way-forward.
Duplicates across lenses are merged into one line, naming each lens that raised it.

## Findings

- forced-edits, BLOCKING — card 4 leaves `fixer_timeout_min` unclassified in `gateslot.ConfigSpec`: fixed as proposed; card 4 edits `internal/gateslot#ConfigSpec`, classing the key `CategoryCrossPair` as `busy_quit_sec`.
- forced-edits, BLOCKING — card 5 leaves the three new keys unclassified: fixed as proposed; card 5 edits `darnengine#ConfigSpec`, `loomengine#TapestryConfigSpec` and `loomengine#ConfigSpec`, each key `CategoryNone`.
- coverage + way-forward, MEDIUM — card 6 does not say what a failed fixer spawn does, and a closure error would end the row: fixed as proposed; a spawn error returns the inner failure with a line naming the spawn error, so the writer is re-prompted, with a `TestWrapRowGate` row.
- coverage + way-forward, MEDIUM — card 6 leaves a record live after an `Await` error: fixed as proposed; the wait stops the strand and marks the record dead `died`, a failed stop keeps it live and returns the joined error, with rows for both.
- guard-reach, MEDIUM — card 6's fixer entry `Gate` never marks a record it waited on terminal: fixed as proposed; every closure that waited marks the record terminal once its closure returns, with a row.
- coverage + guard-reach, MEDIUM/LOW — card 6's `Expired` compares `SpawnedAt` with the wrap's build time, not the run's last attach: fixed differently; the proposed change was to compare against the attach time, which no seam the wrap holds exposes, so the wrap instead keeps in memory the guids it spawned and answers true for a live record it did not spawn, which is what a fresh driver after an attach sees; `RowFixer.Now` is dropped.
- coverage, MEDIUM — card 6 has no row of a forked failure spawning a fixer: fixed as proposed; a row where `SkipForked` rejects the findings spawns one fixer and re-runs `Forked`.
- coverage, MEDIUM — card 12's card fixer prompt is untested (gate call, card ids, no-parent directive), and D5's body contents are untested: fixed differently; the proposal was to assert each body's edit scope, commit rule and leave-list, which is static stencil prose the Plan-Review rubric's test economy rule rejects as a test target; the card 12 row instead asserts the filled values in the spec handed to `fakeCardFixer` (gate call, `--card` ids, reply path, no-parent directive), and card 8's new `TestFixerSpec` asserts the filled re-check commands, gate call and lens review path.
- coverage, MEDIUM — card 13's `CardFixer` decision is untested: fixed as proposed; `cardFixerOn(gate, slug)` is its own member with `TestCardFixerOn` rows.
- coverage, MEDIUM — card 3's `pairSlug` and its register choice are untested: fixed as proposed; `gatefixer.Registrar` returns nil for an empty owner (a `TestRegistrar` row), and `TestWebsterCLI_PairSlug` (tier1, `hubforge`) pins the pair and prime answers.
- coverage, MEDIUM — card 8's unknown-alias refusal and `pairSlugOf` have no row: fixed as proposed; `TestGateFixerSeam` gains the refusal row and `TestPairSlugOf` (tier1, `hubforge`) pins the pair and prime answers; its read error is a propagated raw I/O error, exempt from a reaching test.
- row-validity, MEDIUM — card 8's `TestGateFixerSeam` cannot see the spawn spec inside the wrap: fixed as proposed; the spec builder is split into `fixerSpec` with its own `TestFixerSpec`.
- row-validity + way-forward, MEDIUM — card 12's fixer phase has no lease or signal watch, and its cancel row cancels the context directly: fixed as proposed; `watchPhase` sets up `catchTerminatingSignals` and `cancelOnPassedLease` around the attach wait and the fixer phase, and the row lapses the lease with `expireLease`.
- row-validity, MEDIUM — card 12's `TestRunWithFixer` lacks a live lease fixture and needs git for `tree.json`: fixed as proposed; `writeLiveLease` and `writeRequest` join `Uses`, and `tree.json` is written through a `treeIdentity` seam the test scripts, keeping the test at tier 0.
- way-forward + guard-reach, MEDIUM — card 12 cannot stop a live fixer when no kit is built: fixed as proposed; `newFixerStopper` builds the stop over the worktree's reed geometry whatever `handsOff` answers.
- guard-reach, MEDIUM — card 12's attach wait does not write the `fixing` phase, so the join exemption misses it: fixed as proposed; the phase is written before the attach wait.
- guard-reach, MEDIUM — `shedengine.Relaunch` moves a run past card 9's goto hook: fixed as proposed; `goDriverDepsFor`'s `ConsumeTackler` and `loomConsumeAttempt` call `endRunFixers` before they consume.
- forced-edits, MEDIUM — a `gate-fixer` strand's keep-warm cap would fall back with a warning: disputed, premise false; `keepWarmCapFor` (`internal/shuttleengine/claudeengine/command.go:207-212`) answers `keep_warm_cap` for a role `keep_warm_cap_roles` does not name, with no error, so `KeepWarm` takes no fallback.
- way-forward, LOW — card 3's `Register` failure names no way forward: fixed as proposed; `FixVerifySpawnOrAttach` returns any `gatefixer.SpawnOrAttach` error with the transient re-run way forward, and the row asserts it.
- way-forward, LOW — card 9 does not say what a slug read error does: fixed as proposed; `endGateFixers` refuses it with `transient, re-run the goto`, with a row.
- way-forward, LOW — card 12 marks a record dead even when the stop fails: fixed as proposed; it marks dead only after a successful stop, and a failed stop ends the call with exit 1, leaving the record for the next run to retry.
- guard-reach, LOW — card 12 writes `fixing` after the spawn: fixed as proposed; the phase is written before the spawn.
- row-validity, LOW — card 10's "no `lym loom validate-plan`" assertion would fail on the kept dispute-evidence phrase: fixed as proposed; the rows name the three loop lines that must be gone.
- row-validity, LOW — card 12's not-fixing `TestJoinable` row repeats the existing changed-dirty-paths row: fixed as proposed; the row is dropped.
- row-validity, LOW — card 7's seam-call row repeats the three constructor tests: fixed as proposed; `TestGateFixerElement` keeps its refusal, empty-value and nil-seam rows.
- row-validity, LOW — card 2 lists no stencils fixture: fixed as proposed; `stencilkit.Seed` joins `Uses`.
- row-validity, LOW — card 1's `TestRegistrar` cannot read back a fake pid through `runproc`: fixed as proposed; the row reads the record file directly.
- coverage, LOW — card 10 has no chair row: fixed as proposed; `TestDiscussionTable` gains an advisors row asserting the chair's prompt.
- coverage, LOW — card 1 misses start-failure, engine-lookup and bad-`SpawnedAt` rows: fixed as proposed; rows added.
- coverage, LOW — card 2 misses the friction and `report_path`-overwrite rows: fixed as proposed; rows added.
- coverage, LOW — card 6 rows miss `Final`'s marking and folding, the wait-only dead note, the kept fields, the `Terminal` passthrough and the running-then-terminal wait: fixed as proposed; rows added.
- coverage, LOW — card 7 misses the nil-`Setting` refusal: fixed as proposed; row added.
- coverage, LOW — card 9's `specFor` wiring and no-location receiver have no row: fixed as proposed; `TestSpecFor_PreGotoEndsFixers` added.
- coverage, LOW — card 12 misses the empty-role and `cardFixer`-error rows: fixed as proposed; rows added to `TestHandsOff` and `TestRunWithFixer`.
- coverage, LOW — the deadline bound is argued without a row: disputed, every fix worse; the bound follows from `Final` spawning nothing, which card 6's `Final` rows pin, and a row timing a deadline would need a wall clock the gate loop does not inject (`internal/shuttleengine/wait.go:504-505`, the deadline is `Start`'s), against `PATTERN-test-speed`'s injected clocks.
- forced-edits, LOW — `tools/tokencount` has no `gate-fixer` mapping: disputed, premise false; `partOf` (`tools/tokencount/segment.go:109`) places any unmatched role in the Other segment labelled by its own name, so `gate-fixer` runs are counted apart without a table entry.
- forced-edits, LOW — webster text names `verify-fixer` as a literal: fixed as proposed; card 3 edits `writerRecovery` and the refusal in `RecoverSpawnOrAttach` to use `gatefixer.RoleVerifyFixer`.
