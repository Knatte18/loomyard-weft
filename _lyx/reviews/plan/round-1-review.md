---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: _lyx/plan/17-burler-review-parse-gate.md
    summary: The review gate entry is appended only in Engine.Run, so a live round the adapter re-attaches to through probeLiveRound is not gated on ParseReview, contrary to D12's every round.
  - id: F2
    severity: MEDIUM
    class: scope
    location: _lyx/plan/14-loom-start-never-attaches.md
    summary: D13's doc sweep misses three places that still describe the terminal handover - the LoomBootstrapLock comment, the CheckSeed comment in loomengine/seed.go and the sandbox suite fixture note.
  - id: F3
    severity: LOW
    class: scope
    location: _lyx/plan/15-orch-start-never-attaches.md
    summary: After cards 14 and 15 the reedengine helpers OwnsTmuxEnv, ClientSession and SwitchClientArgv have no production caller, but D13 says the attach code paths go and no card removes them.
  - id: F4
    severity: LOW
    class: design
    location: _lyx/plan/10-compaction-boundary-turn-ends.md
    summary: ReadTurnEndAfter rests on the Stop payload's last_assistant_message equalling the transcript's final text block, an assumption the card states as fact and neither normalizes nor records beside the decision record's assumptions.
  - id: F5
    severity: NIT
    class: consistency
    location: _lyx/plan/02-claude-input-box-reader.md
    summary: The card plans a test pinning SubmitSettle to its own constant, a tautology; Card 3's sleep-order assertions already cover the settle's use.
---

### [MEDIUM] Review gate misses the re-attach hop

**Location:** `_lyx/plan/17-burler-review-parse-gate.md`, against `internal/shedadapters/burler.go` `probeLiveRound`.
**Issue:** D12 adds the parse entry to every round's shuttle gate.
Card 17 appends it inside `Engine.Run` only.
`probeLiveRound` attaches to a still-live round with `AttachGated(spec, p.opts.Gate)`, and `p.opts.Gate` never carries the entry, so a round resumed after a driver death can finish with an unparseable review that nothing re-prompts.
The adapter's own comment says "one GateSpec at every hop", and the card breaks it.
**Fix:** Card 17 creates `ReviewGateEntry(reviewPath)` as the one place the entry is built, `Engine.Run` appends it, and `probeLiveRound` appends the same entry to the gate it attaches with.
`ReviewGateEntry` is described in the card's prose and declares no `plan:` handle, since no other card uses it.
The card adds `internal/shedadapters/burler.go` and extends the existing `TestBurlerProducer_Gate_ProbeLiveRoundPassesGateAndMapsFailedGateIdentically`.

### [MEDIUM] Incomplete doc sweep for D13

**Location:** `_lyx/plan/14-loom-start-never-attaches.md`.
**Issue:** D13 and the acceptance criterion "No doc, stencil or help text describes a start verb handing over the terminal" cover every place that describes the handover.
`internal/loomengine/config.go` (`LoomBootstrapLock`: "released before the terminal handover because that handover blocks for the operator's entire session"), `internal/loomengine/seed.go` (a comment on "the tmux handover") and `tools/sandbox/SANDBOX-CORE-SUITE.md` (the fixture note naming `lyx loom start`'s "tmux bootstrap handover") still describe it, and no card lists them.
**Fix:** Card 14 edits the three and rewords each to the success-envelope behavior.

### [LOW] Dead attach helpers left in reedengine

**Location:** `_lyx/plan/15-orch-start-never-attaches.md`, against `internal/reedengine/client.go`.
**Issue:** `OwnsTmuxEnv`, `ClientSession` and `SwitchClientArgv` are called only by the two start verbs' handover code, which cards 14 and 15 delete.
`AttachArgv` and `TmuxPath` stay live through `lyx reed attach`.
**Fix:** Card 15 deletes the three helpers from `client.go` and the tests that pin them in `client_test.go`.

### [LOW] Unrecorded matching assumption in the turn-end reading

**Location:** `_lyx/plan/10-compaction-boundary-turn-ends.md`.
**Issue:** The card decides "the turn end the caller read is the newest" by comparing the Stop payload's `last_assistant_message` with the transcript's final text block.
It assumes the two are byte-equal.
A whitespace difference between them makes the watcher type no reload, silently and permanently.
**Fix:** The card compares the two with surrounding whitespace trimmed, and states that a mismatch fails closed (R3) and that the equality is an assumption the implementer verifies against a real transcript.

### [NIT] Tautological constant pin

**Location:** `_lyx/plan/02-claude-input-box-reader.md`.
**Issue:** "`SubmitSettle` is pinned to the constant" asserts a value equals its own definition.
**Fix:** The card drops the pin; Card 3's `Sleep:<settle>` log rows already prove the settle is used.

## Fidelity and the other focus questions

- D1 to D3 sit in cards 1 to 4, D4 in 5, D5 in 6 and 7, D6 in 8, D7 in 9 (the history is listed whole, so no omission count is needed), D8 in 10 and 11, D9 and D10 in 12 and 13 with the `doc.go` update in 13, D11 in 16, D12 in 17 and 18 with one `ParseReview` reached by gate, verb and post-gate parse, D13 in 14 and 15, D14 in the shared decision.
  Every Constraint is carried.
- Attack surface: cards 3, 4, 5, 6, 7, 8, 16, 17 and 18 each state what they can skip or remove and what bounds it.
- Verify coverage: every targeted package's tests run under `go test ./...`, `-tags integration ./...` or the `tmux` line, which names the only targeted packages with `tmux`-tagged files, `loomcli` and `webstercli`.
- Test economy: cards 3, 6, 7, 14, 15, 16 and 17 extend existing tests or scenarios, as named; Card 15's one new test is justified by the process-global `TMUX`.
- Granularity: the 10/11, 12/13 and 17/18 splits follow independently reviewable units.
- Scope: the only new verb is D12's `validate-review`; card 15 leaves the orch resume verb alone.
