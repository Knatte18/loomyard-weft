# Round 2 fixer-report — discussion

Every finding's premise held against the code (`internal/boardengine/replay.go`, `internal/battencli/resumereconcile.go`, `internal/pairteardown/originseams.go`, `internal/fabricengine/replay.go`, `internal/shedadapters/roundusage.go`, `internal/boardengine/parentbeam.go`, `board.go`, `spawn.go`, `sync.go`, `dates.go`, `store.go`, `internal/shuttleengine/sessionusage.go`, `transcriptread.go`, the Codex bench evidence).

## Changes

- F1 (fixed as proposed): D9 step 3 no longer marks the run's entry done and leaves it open; a new step 7 marks it done and clears `notify_on_land` as the run's last write. Step 3's done marker is the raddle store directory written in the same write, carrying the approved proposal's hash. D9's Bound states why the entry stays open, and a halt before step 7 shows on the entry and resumes; the old halt-notice clause and its criterion are dropped. New Constraints Fact on the origin paths a done entry meets (`FabricReplay.Runs`, `resumeRefusal`, `teardownEnded`, `withoutFinalizeEntered`); the run-lock Fact now points at step 7. D9 retry criterion rewritten for the store marker. The support log's rejected alternative replaced with why the final done write now works; Q8, Q19 and the Auto-mode assumptions updated.
  Fixed differently in one part: D8's free-slug check gains no skip for this run's own slugs, because D9 runs no re-check once step 3's marker is there, so the skip would never be reached.
- F2 (fixed as proposed, keep-and-narrow option): D12 says the round usage record keeps its own per-half token reading, which no cost report reads. The `PATTERN-run-usage-record` text binds tokens and cost per session under `_lym/usage` and forbids cost reports from reading a round usage record. New D12 criterion. Rejected alternative and Q22 added.
- F3 (fixed as proposed): the `beam` shape's parts are filed unreleased in step 3, and a new step 5 releases them after step 4 made the project a beam. New D9 criterion. Rejected alternative (setting the beam flag in step 3) and Q20 added.
- F4 (fixed as proposed): D10 states that the step 3 write spawns the board sync, whose commit stages every Bolt change, store included, and that `FabricReplay` claims the store's paths. New Constraints Fact citing `boardCriticalSection`, `spawnSync`, `commitDirty`, `Bolt.Commit` and `Claims`. D10 criterion covers a concurrent board write from another machine.
- F5 (fixed as proposed): D8 bullet 2 has no time condition and excludes the run's own entry; D8's baseline wording is removed. D6 gains the check that `board_checked` holds every such entry. The D8 criterion covers the promote case.
- F6 (fixed as proposed): "every filed slug exists" is dropped from step 3's done marker (see F1).
- F7 (fixed as proposed): D12's writer reads messages, forks included, through the engine's `TranscriptReader` from `shuttleengines.RecordReaders` and prices each at its own model. An engine with no `TranscriptReader`, Codex, takes counts from `SessionUsage` with cost unknown. `model` is the session's own transcript's model, and a multi-model session also records each model's counts and cost. The D12 criterion is extended with a fork on another model.
- F8 (fixed as proposed): new D6 check that every decision id a carried criterion or risk names is built by the part or a part on its `depends_on` path. Added to the D5 and D6 criterion.
- F9 (fixed as proposed, keep-the-slug-key option): D5, D6 and D8 require filed slugs free in the raddle store too. D2's start guard refuses a Sketch whose slug the store holds as another Sketch's (another `created` stamp) or as a filed slug. D10 states the store stays keyed by slug and records the `created` stamp that identifies a run's directory. D2 criterion extended. Rejected alternative and Q21 added.
- F10 (fixed as proposed): settled from `docs/research/codex-astra-context-parity-bench-2026-10-27/data/rollout-evidence.txt` (Codex CLI 0.164.0): `total_tokens` equals input plus output, so `output_tokens` already includes `reasoning_output_tokens`. A new Constraints Fact cites it, and D12 says so.
- F11 (fixed as proposed): D7's Bound has `raddle-approve` refuse until the current Raddle-Write session has a passing `raddle-propose`. Added to the D7 criterion.

## Self-check

- fix-claims MEDIUM / fix-proposal MEDIUM (merged): the design-file rule's plain-file-name check also refuses store paths. Fixed as proposed: D10 has the replay admit store paths to both `Claims` and the design-file rule. The new Fact names `applyDesignFile`.
- fix-claims LOW: "spares only a run that entered Finalize" is too strong. Fixed as proposed: D9 Bound and the Fact name the prime's done or Land record too.
- fix-claims LOW / fix-consistency MEDIUM (merged): the cited evidence shows `cache_write_input_tokens`, contradicting "no cache-write count"; "every rollout total recorded" overreached. Fixed as proposed: the Codex Fact says some totals carry `cache_write_input_tokens` (zero) and others none. D12 and its criterion read it where present, zero otherwise. The output Fact is narrowed to the context-parity bench's totals.
- fix-consistency LOW: D9 step 1 did not stage the `created` stamp. Fixed as proposed.
- fix-consistency LOW: D9's opening did not say what makes a store directory the run's. Fixed as proposed: D9 and D10 key it by slug plus the entry's `created` stamp.
- fix-proposal MEDIUM: a replay can land the store marker while refusing a filed entry's add. Fixed as proposed: D10's replay sets aside a store directory's files when it refuses or loses any entry the directory records as filed. The D10 criterion covers it.
- fix-proposal MEDIUM / fix-way-forward outside-lens note (merged): D2's guard checked only Sketch directory names, not filed slugs. Fixed as proposed: the guard also refuses a slug any stored proposal filed. D2, D10 and the D2 criterion are updated.
- fix-proposal LOW: the D6 `depends_on` path check forces serialization. Disputed, premise false (see below).
- fix-proposal LOW: the "Filing parts as drafts" rejected alternative read as contradicting the beam shape. Fixed as proposed: it now notes the beam parts' unreleased window.
- fix-way-forward: no findings.

## Disputed

- fix-proposal finding 4 (self-check, LOW), ground `premise false`: it claims the D6 check forces dependencies between parts that need not depend on each other. A criterion naming two decisions can pass only once both are built, so the part that carries it does need the other decision's builder before it. This is review F8's premise ("a part can be filed with a criterion it cannot meet"). Only the carrying part waits; the other builders stay parallel.

## Deferred

None.
