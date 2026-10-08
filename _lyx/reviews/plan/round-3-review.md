---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: decision
    location: _lyx/plan/09-resume-circling.md
    summary: Two ways forward over an awaiting Bouncer row stay unedited and are not recorded as exceptions to D6, the circling verbs' malformed-escalation way forward (circlingWayMalformed, refusal-spec row circling escalation record malformed) and batten's decide-again hint in innerrun.go awaitingStep
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/plan/08-fix-start.md
    summary: Card 8 creates a new unexported helper in internal/burlerengine/handoff.go that both start orders call, but declares no plan handle for it, against the writer contract that every new member a card creates is declared
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/plan/09-resume-circling.md
    summary: Card 9 leaves the resume descriptions that say resume refuses every awaiting or non-halted state unedited, docs/overview.md's loom resume sentence and the resume.go file header, although the card makes resume wake a driver over an awaiting Bouncer row
---

Round 3 judged the plan against `_lyx/discussion/decision-record.md` (generation 0: `_lyx/loom/rework/` does not exist, so the decision record alone is the answer key).
`lyx loom validate-plan` exits clean (informational `create-new-unit` only), and `lyx quarry resolve` answers `found` for every member glyph the cards name.

Verdicts on the three focus questions:

1. **Edit-card member glyphs and re-sign arrows.**
   Every non-test Go file an `Edit` group targets is either a member glyph or a `doc.go` whose edit is the package doc, which has no symbol to name.
   The remaining file-level edits the Intents describe (the headers of `append.go`, `config.go`, `circling.go` in both packages) are file headers outside any member, and listing the file beside its member would be refused as `redundant-file-target`.
   No card re-signs a member: cards 2, 7, 8 and 9 each state the members keep their parameters and results, and card 3's two new functions carry declaration heads on their `Create` handles.
   No file-only target is left where an Intent names a symbol.
   The one gap in this area is the converse, a new member with no handle (F2).
2. **Card 9's `PendingCirclingDecision` states.**
   No decision file: refused naming `lyx loom circling accept|continue <slug>` then `lyx loom resume`, and both circling verbs accept a run awaiting at an escalated Bouncer row, so nothing refuses back.
   Pending accept or continue: the halted path, whose no-live-driver refusals name `lyx batten run <slug>` (batten's `awaitingStep` revives a dead driver) or `lyx loom start` (which spawns or wakes a driver over an awaiting run in `runHaltedAtHandBack`); both move the run.
   Settled accept: `settleCirclingAccept` runs inside a step whose failures return an error, not `Awaiting`, and the next entry's `settledGeneration` archives the round, so the card's argument that it never sits under `awaiting` holds.
   Run-directory and report-file read errors (`ResolveRound` names the path) and decision-file errors (wrapped with the path) share the unreadable refusal, whose fix-or-delete way forward ends at the circling verbs, which `RecordCirclingDecision` accepts once the file is gone.
   Each refusal has a refusal-spec row and a `resume_test.go` row; the run-directory error shares the decision-file refusal's message and row, and the malformed-file row reaches that path.
   Not a defect.
3. **Unedited operator-facing text over an awaiting Bouncer row.**
   `awaitingHandOff` in `internal/battenshed/innerrun.go` is edited by card 9, so the directive's example is covered.
   Two other ways forward over such a row are neither edited nor stated as exceptions (F1).

### [LOW] Two awaiting-Bouncer ways forward left outside D6 without a stated exception

**Location:** `_lyx/plan/09-resume-circling.md`; `00-overview.md` Shared Decision `halted-paths-keep-start`; code at `internal/loomcli/circling.go:40` (`circlingWayMalformed`), `contracts/specs/refusal-spec.md` row `circling: escalation record malformed`, `internal/battenshed/innerrun.go:764`.

**Issue:** D6 requires every way forward over an awaiting Bouncer row to name `lyx loom resume` and the circling verbs, never approve or reject.
Two such texts are left as they are, and the plan does not say why:

- `circlingWayMalformed`, the circling verbs' refusal over a malformed escalation record, names `lyx loom start`, which re-escalates the round.
  It is emitted only while the run awaits at a Bouncer row.
  `lyx loom start` is the right verb there, since `lyx loom resume` with no pending decision refuses, but `halted-paths-keep-start` lists only the Bouncer's Stuck-path refusals and the carry-over refusal as exceptions.
- `awaitingStep`'s decide-again hint ("the <kind> at <at> was already acted on and the child is awaiting again; re-run lyx loom approve or lyx loom reject") fires whenever an acted-on approval or rejection record is still present and the child awaits again.
  A rejection is cleared by `PR-Rework`, but an approval record is removed only by `lyx loom reject`, so a child that a `goto` sends from an approved PR-Gate back into a review segment that then escalates gets this hint at a Bouncer row, and `lyx loom approve` there refuses.
  Card 9 calls `awaitingHandOff` "the only batten change" without settling this second hint.

**Fix:** Add `circlingWayMalformed` to `halted-paths-keep-start` as a stated exception (start re-escalates; resume would refuse), and either extend card 9's batten edit to the decide-again hint (naming the circling verbs and `lyx loom resume` beside approve and reject) or record it in the same Shared Decision as an accepted exception, with its reachability.

### [LOW] Card 8's shared round-failure helper has no plan handle

**Location:** `_lyx/plan/08-fix-start.md`, Intent bullet on `join`.

**Issue:** The card moves the round's failure rules out of `Engine.join` "into one unexported helper in `handoff.go` that both orders call".
That is a new member in an existing file, outside `join`'s span, and the card's `Create` group declares only `FixStart` and its two constants.
The writer contract says every new member a card creates is declared as a `plan:` handle, whether or not a later card references it.
Without it, `ScopeGuard` reports the helper as `scope-outside-plan` at record-batch, and no done-check confirms it was created.

**Fix:** Name the helper and add a `Create` bullet `plan:internal/burlerengine#<helper>` -> `<declaration head>` to card 8.

### [LOW] Resume's own descriptions still say it refuses every awaiting state

**Location:** `_lyx/plan/09-resume-circling.md`; code at `docs/overview.md:387-388` and the file header of `internal/loomcli/resume.go`.

**Issue:** Card 9 makes `lyx loom resume` wake the parked driver over an awaiting Bouncer row with a pending decision.
`docs/overview.md` still says resume "wakes a halted run's live, parked driver … and refuses every other state", and `resume.go`'s header says it wakes a halted run's driver and refuses "everything else".
Card 9 lists the overview's escalation and batten paragraphs and the verb's `Long` table, but not these two sentences, so both go stale in the commit that changes the behavior (CLAUDE.md: docs land in the same commit).

**Fix:** Add both to card 9: the overview sentence and the `resume.go` header name the awaiting-at-a-Bouncer-row case with a pending circling decision beside the halted case.
