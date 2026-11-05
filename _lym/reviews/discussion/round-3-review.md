---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md:158
    summary: "D10 says no slug in the raddle store is reused, but only a raddle run is refused a reused slug, so a later unrelated entry with a stored slug is matched to the old proposal by the slug lookups of D10, D11 and D14"
    fix: |
      In D10, replace the sentence "The store stays keyed by slug, and no slug in it is reused: D2 refuses ..., and D5, D6 and D8 require every filed slug free in the store as well as on the board." with a sentence that adds the board-wide rule:
      the board's new-slug rule (`checkNewSlugs` in `internal/boardengine/slugrule.go`, which already judges every write that creates a task or project, renames an entry or makes a note a task) also refuses a slug the raddle store holds as a Sketch's slug or as a filed slug, naming a different slug as the way forward;
      the entry whose `created` stamp the store records for its own Sketch slug is exempt, so a live raddle run's Sketch is never refused.
      Keep D2's start guard and D5, D6 and D8's free-slug checks as they stand.
      State in D10 that `lym board raddle <slug>`, D11's lookup of the store by the beam project's slug and D14's listing rely on this refusal, so a slug in the store always names the entry that filed or was raddled under it.
      Add an acceptance bullet under D10: `lym board upsert` creating a task or project, `lym board rename` and a note promotion under a slug the raddle store holds are each refused with a different slug named, and after a filed project is pruned a new project of the same slug is refused rather than matched to the old proposal.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md:144
    summary: "D9 step 5 uses the released parts as its done marker, which a retry cannot read once a released part has landed and any run's Land has pruned it, the failure D9 already avoids for step 3"
    fix: |
      In D9 step 5, replace "the parts released are its done marker" with:
      the write releases each filed part that is still on the board and unreleased, and skips a part that is absent from the board or done, since a released part can land and a Land prune it before a retry;
      the step's done marker is that no filed part still on the board is unreleased.
      Add to the D9 acceptance bullet on interrupted retries: a Raddle-File interrupted after step 4 or after step 5 and run again once a released part has landed and been pruned releases the parts still on the board, skips the pruned one and goes on to steps 6 and 7.
  - id: F3
    severity: LOW
    class: design
    location: _lym/discussion/decision-record.md:233
    summary: "The Open risk on pawl-orch-landing leaves a researchable fact to Plan, and D11 does not say whether BeamUp mails before or after it records the sent marker"
    fix: |
      Delete the Open risks bullet "Task `pawl-orch-landing` gives BeamUp a re-invoke policy; D11's sent-marker must hold under it."
      Add a Fact to Constraints:
      the board entry `pawl-orch-landing` gives Beam-Up the re-invoke interrupt policy, since the row spawns no agent and is idempotent (its D11, stored with that entry on the board).
      In D11, state the order: BeamUp mails the hand-off, then records it as sent in its pawl run's state, so an interruption between the two lets a re-invoked BeamUp mail a second identical hand-off, which D11 accepts, and a BeamUp re-invoked after the marker is recorded sends none.
      Replace the D11 acceptance clause "a re-invoked BeamUp ... send none" with the same two cases.
  - id: F4
    severity: LOW
    class: design
    location: _lym/discussion/decision-record.md:179
    summary: "D12's usage writer runs at every session end of every run but states no rule for a failed locate, a failed transcript read or a failed write"
    fix: |
      In D12, after the sentence that has the writer locate the session and read its messages, add:
      a locate or read failure, such as a transcript that is gone or unreadable, never fails or stops the session or the run;
      the writer logs it at Warn and writes the session's file from `SessionUsage`'s four counts with the cost unknown, as it does for an engine with no `TranscriptReader`, and a failed write of the file is logged and stops nothing.
      Add to the D12 acceptance bullet on session files: a session whose transcript cannot be read still writes its usage file from `SessionUsage`'s counts with an unknown cost, and its run goes on.
---

### [MEDIUM] D10's no-reuse invariant holds only for raddle runs

**Location:** `_lym/discussion/decision-record.md:158` (D10), with D2 at line 37, D11 at line 168 and D14 at line 195.
**Issue:**
D10 states that no slug in the store is reused, and enforces it in three places: D2's batten start guard, D5's and D6's free-slug rule for filed slugs, and D8's re-check.
All three act only on a raddle run's own Sketch or filing.
The board's new-slug rule that judges every other creator, `checkNewSlugs` in `internal/boardengine/slugrule.go`, runs only the pair slug seam `Config.ValidateSlug` and knows nothing of the store.
Prune frees the slugs of the Sketch and of every landed part and project, and the store, which `Store.Prune` never touches, keeps them.
A later `lym board upsert` of a project under an old filed project's slug passes every board check.
D11 then finds "a proposal that filed the beam's project" by slug alone and mails the new beam's orch a hand-off for the old Sketch, and `lym board raddle <slug>` and D14 print the old proposal beside the new entry.
D10 records filed slugs without their entries' `created` stamps, so no lookup can tell the old entry from the new one the way D10's run rule tells a Sketch by its stamp.
The invariant D11 and D14 rest on is therefore not enforced.

### [MEDIUM] D9 step 5's done marker disappears when a released part lands and is pruned

**Location:** `_lym/discussion/decision-record.md:144` (D9 step 5), with the retry acceptance bullet at line 253.
**Issue:**
D9 makes the raddle store step 3's marker because "a released part can land and be pruned before a retry" (support log, Rejected alternatives).
Step 5 reads "the parts released are its done marker", which fails the same way:
step 5 releases the parts of a `beam` project, and from then on the beam's dobby can start them.
A part that lands is pruned by the Land of any run on the board, since `Store.Prune` removes every done entry not carrying `notify_on_land`.
A halt between steps 5 and 7 waits for the parent, so the retry can come after that.
The retry then finds a filed part absent, and D9 says neither what step 5 does with it nor how the step reads as done.
The acceptance bullet "interrupted after any step and run again finishes the remaining steps and repeats none" cannot hold for this case as written.

### [LOW] An Open risk defers a researchable fact, and D11's marker order is unstated

**Location:** `_lym/discussion/decision-record.md:233` (Open risks) and D11 at lines 166 to 170.
**Issue:**
The Open risk "Task `pawl-orch-landing` gives BeamUp a re-invoke policy; D11's sent-marker must hold under it" leaves to Plan a fact the writer could read.
The peer's stored record, `decision-pawl-orch-landing.md` in the Bolt, gives Beam-Up the re-invoke interrupt policy in its D11, "since each spawns no agent and is idempotent".
D11 adds a mail to BeamUp, which ends that premise, and it names the sent-marker without saying whether the mail or the marker comes first.
A crash between them either loses the hand-off, if the marker comes first, or sends it twice, if the mail does.
The writer can choose the order and record it, so no parent decision is needed and this is not a `decision` finding.

### [LOW] D12's usage writer has no failure rule

**Location:** `_lym/discussion/decision-record.md:179` (D12).
**Issue:**
D12 puts one writer at every session end of every run.
It reads a transcript through `TranscriptReader`, whose `ReadSession` "errors when a transcript cannot be read, a vanished one included" (`internal/shuttleengine/transcriptread.go`).
D12 names the unknown cost for an unpriced model and the Codex fallback, and says nothing for a locate or read error or a failed write.
Without a stated rule, a plan or implementer can let a recording error halt a session or run of any recipe, a regression for a figure only reports read.

## Rejected

- Focus 1, resume after step 3 reaches origin: not a defect.
  D9 leaves the run's entry open until step 7, and `FabricReplay.Runs` sets `Ended` only from `isDone` (`internal/boardengine/replay.go`), so a halt or crash at step 4 or 5 meets none of `resumeRefusal`, `teardownEnded` or `withoutFinalizeEntered`.
  Step 3 leaves the store directory carrying the approved proposal's hash, and the run's entry becomes done only in step 7, as D9 and its Bound state.
- Focus 2, marker against prune: not a defect for step 3.
  The step 3 check reads the store directory and the run's own entry's `created` stamp, neither of which `Store.Prune` removes while the run's entry is open.
  Step 5's marker is the defect, filed as F2.
- Focus 3, release order for shape `beam`: not a defect.
  D9 files the parts unreleased and releases them in step 5 after step 4's `lym beam create`, so `BeamUnder`'s fall back to `ParentBeamOf` while the project is no beam cannot start a filed part.
  `lym beam create` sets the beam flag in its owner write (`internal/beamcli/create.go`), before step 5.
- Focus 4, staging and replay of the store: not a defect.
  `commitDirty` calls `Bolt.Commit`, which calls `StageAllAndCommit`, so the store directory reaches origin in the board sync's commit.
  `FabricReplay.Claims` and `applyDesignFile` refuse a path with a `/` today, as D10 and its Constraint say, and `replayBolt` sends a commit holding any unclaimed path to cherry-pick, which conflicts on `board.json` when another machine wrote concurrently;
  the claim D10 adds is therefore what makes the concurrent write replay.
- Focus 5, D12 against the declared `PATTERN-run-usage-record`: the declared text matches D12.
  The entry binds per-session tokens and cost under `_lym/usage` to one writer and forbids a cost report from reading a review round's usage record.
  `internal/shedadapters/roundusage.go` writes its per-half reading to `round-<N>-usage.yaml` under the review directory, outside `_lym/usage`, and D12 and its acceptance bullet keep it as a summary no report reads.
- Focus 6, pricing source: not a defect.
  D12 prices from messages read through `TranscriptReader` and `MessagePricer` from `shuttleengines.RecordReaders`, not from `SessionUsage`'s totals, which it uses only for an engine with no `TranscriptReader`.
  `model` is the session's own transcript's model, and a fork on another model is covered by the per-model counts D12 requires.
  The failure path of that read is F4.
- Focus 8, D6 and D8 checks: not a defect.
  D6 checks every decision id a part's carried criterion or Open risk names against the part and its `depends_on` path.
  D8's membership rule, an open task or project absent from `board_checked` that carries a module label "whenever it was created or promoted", covers an entry created after the agent's read and a note promoted with its old dates, since it reads no date.
- D9 and D2 filing and promoting: not flagged under the rubric's file-or-promote bullet.
  The entry asks for the filing, D7 binds it to the operator's approval or the operator's auto key, and D9's Bound states that the board's refusal of a run agent's task creation still holds for the agent's own `lym board` calls.
