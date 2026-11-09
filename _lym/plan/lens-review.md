# Lens review

Every lens fork returned a report: coverage, row-validity, forced-edits, guard-reach and way-forward.
Duplicates across lenses are merged into one line, naming each lens.

- coverage, row-validity, guard-reach, way-forward — MEDIUM — `Command`'s `RunE` branches (follow spawns nothing, exit codes, `NoOrch` from an orch strand, `--discard` misuse) have no reaching test — fixed as proposed: card 8 adds `runAdopt` over a `callerDeps` seam with a fake `Spawn`, and `TestRunAdopt` has one row per branch.
- coverage — MEDIUM — `follow`'s "a record not started later is not followed" has no row — fixed as proposed: a `TestFollow` row for an earlier-started record of another pid.
- coverage — MEDIUM — a held-back project ending the record `stopped` has no record-level row — fixed as proposed: a `TestRunWorker` row asserts `stopped` at `done`, the way forward naming `--keep-local` and `--drop-local`, and the other subject reaching `suite:`.
- coverage, guard-reach — MEDIUM — a failed pause update has no row and no failing fake — fixed as proposed: card 6 adds `world.pauseErr` and a `TestAdopt` row stopping at `pause` with no `owners` in the order.
- coverage — MEDIUM — the green end's "no lift, no start, no resume" is argued by construction — disputed, every fix worse: after card 4 `greenDeps` holds only the fix-task and tries seams (card 4's field list against `internal/adoptcli#greenDeps`), so a row has no seam through which it could observe a lift, a bring-up, a pass or a resume, and its assertion could never fail (rubric "Test economy"; row-validity's fails-if-wrong test).
- coverage — MEDIUM — the hibernate wake leaving an adopt pause standing has no row — fixed as proposed: card 1 adds a `TestSequence` row with a live `AdoptCaller` entry standing after the wake.
- coverage — MEDIUM — no row shows a converted beam slug in `runs` as `paused_by_adopt` — fixed as proposed: a card 6 `TestAdopt` beam row converts `alpha`'s `owner` entry and reads it in `runs`.
- coverage — LOW — the fresh record written `running` at `reconcile` before any step has no row — fixed as proposed: a `TestRunWorker` row on the first `Save`.
- coverage — LOW — a restart at `reconcile` could list a step twice — fixed as proposed: card 6's `runSteps` replaces a `StepRecord` of the same name, and card 7's restart row asserts each step listed once.
- coverage — LOW — a `beam status` read error of `adopt.json` has no row — fixed as proposed: card 9's integration test writes an undecodable record and asserts the error naming `lym adopt --discard`.
- coverage — LOW — `discard` with no record has no row — fixed as proposed: a `TestDiscard` row.
- coverage — LOW — `followCap`'s keep-warm wiring and empty-strand zero have no row — fixed differently: rather than a test of the hub wiring, `followCap` takes a `keepWarm` func (re-signed handle) and `TestRunAdopt` rows cover the empty strand without a `KeepWarm` call and the agent strand's cap.
- coverage — LOW — `ownerHaltResume`'s empty answer has no exact row — fixed as proposed: a card 5 row compares a blocked-for-another-reason line whole.
- coverage — LOW — the worker's `runproc` registration has no row — fixed as proposed: card 8 states it is held by review per `PATTERN-child-processes`' Enforcement, since `spawnWorker` refuses under test.
- row-validity — BLOCKING — a resume past `owner` meets nil per-slug outcomes in `runBeamSteps` (`internal/adoptcli/adopt.go:914,927`) — fixed as proposed: card 6's `runSteps` seeds an outcome for every slug of `Subjects`, `Inputs.Runs` and `Inputs.Paused` before its first step.
- row-validity — MEDIUM — card 6's second-pause-run row cannot reach the skip without a persisted record — fixed as proposed: the row moves to card 7's `TestRunWorker` over a seeded record, and `TestPauseEntries`' listed-slug row keeps the unit.
- row-validity — MEDIUM — the world's `order` lacks `reconcile` and pair-add entries the resume rows assert — fixed as proposed: card 6 has `Reconcile` and `AddPair` append to `order`, and the resume row asserts no `addpair:`.
- row-validity — MEDIUM — `followDeps` lacks the writer and cap two `TestFollow` rows need — fixed as proposed: `followDeps` holds the cap and the step-line writer.
- row-validity — LOW — card 1's `TestPause` row repeats an existing row — fixed as proposed: cut; `TestPause`'s caller row at `internal/tenderengine/pause_test.go:777-793` already pins `AlreadyPaused` over another caller's entry.
- row-validity — LOW — card 1's `TestResume` row repeats existing coverage — fixed as proposed: cut; `Resume` lifts every live entry of the scope whatever its caller and card 1 changes no tender code.
- row-validity — LOW — `TestWorkerCommand` in a new file repeats `TestDetachSetsSetsid` — fixed as proposed: the file and test are cut; card 8 names `TestDetachSetsSetsid` as the pin of the new session.
- row-validity — LOW — `TestSpawnWorkerStartsNothingUnderTest` repeats `TestStartSuiteStartsNothingUnderTest`'s shape — fixed as proposed: `TestSpawnersStartNothingUnderTest` replaces it with one row per spawner, the old test deleted.
- row-validity — LOW — card 2's skip-orch row needs an expectation column the table lacks — fixed as proposed: card 2 names a `wantNoOrch` column asserting no launch or watcher among the fake starter's calls.
- forced-edits — MEDIUM — `internal/landingshed/finalize_owner.go:130` promises the green end's resume — fixed as proposed: card 4 edits `internal/landingshed#ownerHaltReason` to name the adopt pause and `lym tender resume` then `lym loom resume`.
- forced-edits — LOW — `beamcli` create help and doc, `battenshed/doc.go` and `docs/overview.md` still say the green end starts runs — fixed as proposed: card 4 edits `internal/beamcli#createCmd`, `internal/beamcli/doc.go`, `internal/battenshed/doc.go` and `docs/overview.md`.
- forced-edits — LOW — pauserecord's doc and the `HoldCaller` comment need rewriting — fixed as proposed: card 1's Intent quotes both sentences that go.
- forced-edits — LOW — `holds.go`'s file comment goes stale — fixed as proposed: card 6 rewrites it to the tries file it keeps.
- forced-edits — LOW — `TestRunReports`' check-state rows set a deleted constant — fixed as proposed: card 5 says those rows are deleted with their state.
- forced-edits — LOW — `Command`'s `Long` is stale between cards 6 and 8 — fixed as proposed: card 6 edits `internal/adoptcli#Command`'s step list.
- guard-reach — MEDIUM — `HeldBack` could take a project held back by an error, naming the wrong way forward — fixed as proposed: card 6 fills `HeldBack` only from local-commit or live-pair stops, and a `TestAdopt` row pins a refused pair add.
- guard-reach, way-forward — LOW — `runWorker`'s `follow` verdict cannot occur under the lock — fixed as proposed: card 7 states it as a fail-loud wiring guard with no reaching test.
- guard-reach — LOW — the beam rule of `pauseEntries` collides with an adopted beam's pawl run as a target — fixed as proposed: card 6 says the target rule wins for a beam among the targets.
- guard-reach — LOW — what `follow` gets on the follow verdict is unstated — fixed as proposed: card 8 passes the record's pid and a nil `exited`.
- way-forward, guard-reach — MEDIUM — `--discard` cannot clear an undecodable record — fixed as proposed: card 3 adds `adoptrecord.ErrDecode`, card 8's `discard` removes such a record holding the lock, and `TestDiscard` gains the row.
- way-forward — MEDIUM — a `--keep-local` restart can keep the old `HeldBack` — fixed as proposed: card 6's `stepReconcile` clears every `Inputs` field but `Paused`, and card 7's restart row ends `done`.
- way-forward — LOW — the follow's no-record error names no cause — fixed as proposed: it names `adopt.out`.
- way-forward — LOW — `heldBeamWayForward` names a tender resume with nothing to lift — fixed as proposed: it gains `paused` (re-signed on card 6) and names `lym tender resume <slug>` only when `Inputs.Paused` lists the beam.
