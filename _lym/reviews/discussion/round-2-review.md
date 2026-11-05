---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D9 step 3 and Bound; Constraints Facts on runlock and board.go; D9 acceptance criteria on retry
    summary: Step 3 marks the run's own entry done mid-run, so origin reads the live run as ended; a halt or crash after step 3 cannot be resumed, the pair can be torn down and its later weft commits set aside
    fix: In D9, stop marking the run's entry done in step 3. Make step 3's done marker the raddle store directory written in the same board write, holding the approved proposal's hash, and have D8's free-slug check skip slugs that store records as filed by this run. Mark the run's entry done, with notify_on_land cleared, in a final board write after step 5, or leave it to the run's finish. Add a Constraints Fact naming the origin paths an entry done before the run ends meets (boardengine replay.go Ended is isDone; battencli resumeRefusal; pairteardown teardownEnded; fabricengine withoutFinalizeEntered sparing only runs past Finalize) and update D9's Bound and its retry acceptance criterion to the new marker. Replace the support log's rejected alternative on a final done write with the reason it now works.
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D12 and PATTERN changes (PATTERN-run-usage-record); internal/shedadapters/roundusage.go
    summary: The declared PATTERN allows only shed's one usage writer to record a session's tokens, but roundusage.go already records each review half's token reading and D12 neither folds it in nor exempts it
    fix: In D12, state that the round usage record stops writing its own token readings and takes them from the per-session usage files, or that it moves onto the one writer. If the round record keeps its own readings, narrow the PATTERN-run-usage-record text in PATTERN changes so it binds tokens and cost per session under _lym/usage and names round usage as a reader. Add an acceptance criterion that pins the chosen behavior.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D9 steps 3 and 4
    summary: For shape beam, step 3 files the parts released in a project with no beam flag, so the parent level's dobby can start them under the Sketch's parent before step 4 creates the beam
    fix: In D9, file the beam shape's parts unreleased in step 3 and release them in a step after step 4, with its own done marker, or set what makes the project a beam in step 3's own write. Add an acceptance criterion that no beam-shape part is released before its project is a beam.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D9 step 3 and D10 (raddle store in the one board write and commit)
    summary: A board write commits nothing itself but spawns a detached board sync, and the Bolt replayer claims only root files, so the raddle store directory is neither shown to reach origin with the entries nor replayed entry by entry
    fix: In D10, state which commit stages the raddle store directory (the board sync's pathspec gains it, or the step 3 write commits it) and which replayer claims its paths (boardengine's FabricReplay.Claims, or a replayer of its own). Add a Constraints Fact citing boardengine board.go's spawnSync and replay.go's Claims, with the D10 acceptance criterion covering a concurrent board write from another machine.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D8 bullet 2
    summary: The module-label check sees only entries revised after the latest passing check, so it misses an entry created between the agent's board read and its first passing check, and a note promoted later, since a promote keeps the note's dates
    fix: Rewrite D8 bullet 2 with no time condition. The re-check fails when any open task or project absent from board_checked carries a module label the Sketch carries, and D6 refuses a proposal whose board_checked lacks such an entry. Add the promote case to the D8 acceptance criterion.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D9 opening (step 3 done when every filed slug exists)
    summary: Filed parts are released at once and can land and be pruned before a retry, after which every-filed-slug-exists reads step 3 as not done and re-files or fails the free-slug check
    fix: In D9, drop every filed slug exists from the step 3 done marker and use only the marker the step 3 write itself leaves (see F1).
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D12 (writer prices per message through MessagePricer)
    summary: SessionUsage gives the writer only session totals and one model, while PriceMessage needs each message's usage and model, and D12 names no per-message source
    fix: In D12, state that the writer reads the session's messages through the engine's TranscriptReader from shuttleengines.RecordReaders and prices each with its own model, and say what the record's model field holds when the session and its forks use several models. Extend the D12 acceptance criterion with a fork on another model.
  - id: F8
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D6 checks and D9 step 2
    summary: Nothing requires that a criterion or risk a part carries names only decisions that part or its dependencies build, so a part can be filed with a criterion it cannot meet
    fix: Add a D6 check that every decision id a carried criterion or Open risk names is built by that part or by a part on its depends_on path, and a matching row in the D5 and D6 acceptance criterion.
  - id: F9
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D5 and D8 slug freshness; D10, D11 and D14 store lookups
    summary: Slug freshness is judged against the board, which prune empties, while the raddle store outlives prune and is looked up by slug, so a reused slug collides with an older store entry
    fix: In D5 and D8, require each new slug, and the Sketch's own slug, to be free in the raddle store as well as on the board, or key the store by an id that is never reused and index slugs to it. State which in D10.
  - id: F10
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D12 Codex reading; Constraints Codex fact
    summary: D12 reads Codex output as output_tokens without settling whether output_tokens includes reasoning_output_tokens, a fact the reading rests on
    fix: Settle from Codex's token usage source or rollout samples whether output_tokens includes reasoning_output_tokens, cite the source and version in the Constraints Codex fact, and add reasoning_output_tokens to D12's output if it does not.
  - id: F11
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D7 interactive bullet and D8 return
    summary: After a return in interactive mode, raddle-approve does not require a passing raddle-propose in the new session, so the waiting notice to the parent orch may never go out
    fix: In D7, have raddle-approve refuse until the current Raddle-Write session has a passing raddle-propose, and add that refusal to the D7 acceptance criterion.
---

### [BLOCKING] Step 3 ends the run on origin before steps 4 and 5 run

**Location:** `_lym/discussion/decision-record.md` D9 step 3, D9 Bound, the Constraints Facts on `runlock.go`, `board.go`, `sketch.go` and `store.go`, and the D9 retry acceptance criterion.

**Issue:** The run ledger counts a run ended when its entry is done (`internal/boardengine/replay.go:272`, `Ended: isDone(task)`), so once step 3's board commit reaches origin, the origin reconcile classifies the live raddle run as `EntryEnded`.
After that, each of these existing paths breaks the run:

- A batten full pass, the path any halt or crash recovery takes, refuses through `resumeRefusal`: "origin ended … run lym cleanup --apply … instead of resuming it" (`internal/battencli/resumereconcile.go:114-124`).
- A hub reconcile or `lym cleanup --apply` tears the idle pair down (`internal/pairteardown/originseams.go:155`).
- A weft replay, including the one in Land's own `fab.Pull`, sets aside the run's later record commits, because `withoutFinalizeEntered` spares only runs that entered Finalize and raddle has none (`internal/fabricengine/replay.go:236-251, 305-320`).

A halt at step 4, which D9 plans for, therefore leaves a `beam` shape filed with no beam and no notice, and the criterion "interrupted after any step and run again finishes the remaining steps" cannot pass.
The record's Facts check only the run lock and status writes and never reach these paths.
Its rejected alternative, a final done write after step 5, was dropped only because step 3 then had no done marker, and the raddle store written in step 3's own write gives it one.

### [BLOCKING] A second token writer against the declared PATTERN-run-usage-record

**Location:** D12 and `## PATTERN changes`; `internal/shedadapters/roundusage.go`.

**Issue:** The declared entry reads "Recording an agent session's tokens and cost: only shed's one usage writer".
`roundusage.go` already records each review half's token reading in `round-<N>-usage.yaml`.
D12 cites that file only as evidence that nothing records cost, and never says whether the round record keeps its own reading.
As written, the design leaves two writers of session tokens and so breaks the declared entry's own text.

### [MEDIUM] Beam-shape parts are released before their project is a beam

**Location:** D9 steps 3 and 4.

**Issue:** Step 3 files the parts released inside a project with no beam flag, and only step 4's `lym beam create` makes it a beam.
`BeamUnder` falls back to `ParentBeamOf` while the project is no beam (`internal/boardengine/parentbeam.go:79-82`), so in that window the parent level's dobby can start the parts at the Sketch's level, outside the beam that D11's hand-off serves.

### [MEDIUM] The raddle store's commit and replay are unstated

**Location:** D9 step 3 and D10.

**Issue:** `boardCriticalSection` saves, renders and then only spawns a detached sync (`internal/boardengine/board.go:155-178`), so "one board write and one board commit" that brings the store to origin is not something the write does today.
`FabricReplay.Claims` claims only root files, `board.json`, overviews and `design-`/`decision-` files, and refuses any path with a `/` (`internal/boardengine/replay.go:44-78`).
A commit holding the store directory is therefore not replayed entry by entry, and D10 names no replayer for it.

### [MEDIUM] D8's module-label check misses entries by its time condition

**Location:** D8 bullet 2.

**Issue:** The baseline is the latest passing check, not the agent's board read.
A task sharing a module label that is created between the read and the first passing `raddle-propose` is never compared.
A promote also keeps the note's dates (`internal/boardengine/dates.go:21-47`), so a note promoted to a task after the baseline passes as well.
Both slip past the check the board entry asks for: "the board can move meanwhile, so the proposal is checked against the board again".

### [MEDIUM] Step 3's done marker depends on entries that prune removes

**Location:** D9 opening sentence.

**Issue:** "every filed slug exists" turns false once a released part lands and Land prunes it (`internal/boardengine/store.go:790-796`).
A later retry then reads step 3 as not done, and it either fails the free-slug check on the remaining filed slugs or files the pruned slugs again.

### [MEDIUM] Per-message pricing has no per-message source

**Location:** D12.

**Issue:** `SessionUsage` carries only totals (`internal/shuttleengine/sessionusage.go:7-20`), while `PriceMessage` takes one message's model and usage (`internal/shuttleengine/transcriptread.go:143-147`).
The per-message model and one-hour cache split come only from the transcript parse (`internal/shuttleengine/claudeengine/usage.go:593-617`).
D12 names neither that read nor what `model` means for a session whose forks run on another model.

### [MEDIUM] A carried criterion can name decisions the part does not build

**Location:** D6 and D9 step 2.

**Issue:** D6 checks that each part's criteria cover its own decisions, but not the reverse.
A criterion such as "D5 and D6: …" in this very record can go to a part that builds only D5, and step 2 files it with a criterion it cannot meet.

### [MEDIUM] Slug freshness ignores the raddle store

**Location:** D5, D8, D10, D11 and D14.

**Issue:** Freshness is judged against the board, and prune frees slugs (`internal/boardengine/store.go:790-796`).
The store outlives prune and is looked up by the Sketch's slug, a filed slug or a beam's project slug.
A reused slug therefore overwrites or shadows an older store entry, and D11 can hand a beam an old proposal.

### [LOW] Codex output tokens and reasoning tokens left open

**Location:** D12 and its Codex Constraints fact.

**Issue:** The fact names `reasoning_output_tokens` (`docs/research/codex-review-bench-2026-10-25/report.md:196`), but neither it nor D12 says whether `output_tokens` already includes them.
This is a fact the reading rests on, which the writer's stencil says to settle, not leave for later.

### [LOW] Waiting notice can be skipped after a return

**Location:** D7 interactive bullet, D8.

**Issue:** The notice goes out on a session's first passing `raddle-propose`, but nothing requires that call before `raddle-approve`.
After a return that moved nothing, the new session can approve without proposing, and the parent orch is never told that the run waits for the operator again.

## Focus verdicts

- Focus 1, D9 ordering and retry: the step 1 and 2 outputs are hash-stamped and deleted on a return, so a changed, re-approved proposal rewrites them; that part holds. Every step has a stated marker, but step 3's marker is unstable (F6), and resuming at all after step 3 is refused (F1).
- Focus 2, part records to stored records: D9 step 3 names a new `boardengine` write that creates each entry, stores its record and sets its `discussed` flag in the creating write; that holds. That the raddle store reaches origin in that commit is not shown (F4).
- Focus 3, `raddle_auto_approve`: the mode is read once at Raddle-Write's first spawn and recorded (D7), and a later `--layer local` write changes nothing. Raddle-File runs D6's checks in both modes (D6, D9). Held.
- Focus 4, cost against the engine pricer: D12 to D14 use the engine's `MessagePricer`, Codex's missing pricer leaves its dollars unknown, and its usage fields are recorded with evidence. Two gaps remain: the per-message source (F7), and whether Codex's output includes reasoning tokens (F10).

## Rejected

- bound-breaker F6, D11's marker resends after a cancel and a new pawl run: a new pawl run brings up a fresh beam orch, which needs the hand-off, so sending it again is correct.
