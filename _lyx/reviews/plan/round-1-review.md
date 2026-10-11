---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/01-hook-recording-capture.md
    summary: Card 1 trims the capture into a new testdata/hooks format read only by parser rows, not into claudeengine corpus cases under testdata/corpus that TestCorpus_ReplaysEachTurnEnd replays through the fold, as D1 and the smoke test doc name
  - id: F2
    severity: MEDIUM
    class: scope
    location: _lyx/plan/08-stray-character-clear.md
    summary: Card 8 sends the clear into a box holding a stray character, but leaves InputBoxClearer's doc contract (clear only into a box holding exactly the caller's own text) and Claude.ClearInputSequence's doc untargeted and false
  - id: F3
    severity: LOW
    class: decision
    location: _lyx/plan/08-stray-character-clear.md
    summary: Card 8 does not say how a later poll of the same send attempt reads a box still holding the recorded character once the post-clear reading was held, residue or draft
  - id: F4
    severity: LOW
    class: scope
    location: _lyx/plan/01-hook-recording-capture.md
    summary: Card 1 adds hooks into Claude's permission path and every SessionStart without stating that they print nothing and which alwaysSucceeds value they take, so their bound on the permission decision and the session context is unstated
  - id: F5
    severity: LOW
    class: design
    location: _lyx/plan/02-idle-fresh-fold.md
    summary: Cross-package public surfaces are bundled into their consumer cards, SessionStartSource with its constants in card 2 and StrayInputReader with Claude.StrayInput in card 8, where the granularity rule gives each its own card
---

### [MEDIUM] The hook capture bypasses the existing corpus-case format

**Location:** `_lyx/plan/01-hook-recording-capture.md` (Create targets `testdata/hooks/events.jsonl` and `testdata/hooks/capture.yaml`); carried by cards 2 and 3's `TestParseSessionSignals` rows.

**Issue:** D1 says the captured lines "are trimmed into `claudeengine` corpus cases", and the record's note points at `smoke_turns_test.go`, whose file comment says promoting a kept events file "to a claudeengine corpus case is a copy and a trim".
That corpus exists: `internal/shuttleengine/claudeengine/testdata/corpus/<case>/` holds `case.yaml`, `events.jsonl` and `transcript.jsonl`, and `TestCorpus_ReplaysEachTurnEnd` in `corpus_test.go` replays each case through `ParseEvents`, the wait loop and the session-state fold, checking the `states:` list (the state and cause after each signal-bearing line).
Card 1 instead invents a second fixture layout, `testdata/hooks/events.jsonl` beside a `capture.yaml` of its own shape, and cards 2 and 3 read it only through `TestParseSessionSignals` rows.
So the real lines are checked as parsed signals, but never replayed through the fold.
D1's "every payload-field rule below is tested against them" then holds only for the parser half of D3 and D5: the fold half (the startup line reading idle-fresh, `PermissionRequest` reading asking, `SessionEnd(clear)` then `SessionStart(clear)` reading idle-fresh) is tested only on hand-built signals.
It also leaves two corpus formats under one `testdata/`.

**Fix:** have card 1 write each captured session as a corpus case directory under `testdata/corpus/` (`events.jsonl`, the trimmed transcript the smoke test keeps, and a `case.yaml` whose `states:` list the fold readings, with the gate and registry ordering recorded in that `case.yaml`'s free fields as the existing cases record `what_went_wrong`).
Cards 2 and 3 then extend those cases' `states:` with the idle-fresh and asking readings their rules produce, and point their `TestParseSessionSignals` rows at the case's `events.jsonl`.
Card 1's targets change from the two `testdata/hooks/` files to the new case directory's files, and `corpus_test.go` joins card 1's `Edit:` only if the case's new keys need reading.

### [MEDIUM] `InputBoxClearer`'s contract is left false by card 8's clear

**Location:** `_lyx/plan/08-stray-character-clear.md`, the `**Edit:**` list and the Intent paragraph on playing `ClearInputSequence`.

**Issue:** `internal/shuttleengine/engine.go`'s `InputBoxClearer` doc states the contract every implementer relies on: "The caller sends the clear only into a box it has just read as holding exactly its own text".
`claudeengine`'s `Claude.ClearInputSequence` doc justifies its single `C-u` on the same premise ("Sent text is one line with the caret at its end, so that empties the box").
Card 8 adds a second caller that sends the clear into a box holding a stray-character candidate, which is not the caller's own text, so after card 8 the interface doc misstates when the clear is sent.
Neither symbol is among card 8's targets, and the Intent never mentions the interface's contract.
The `PATTERN-pane-text-injection` background update the card does carry lists the new key delivery, but the interface doc a provider author reads is left contradicting it.

**Fix:** add `internal/shuttleengine#InputBoxClearer` and `internal/shuttleengine/claudeengine#Claude.ClearInputSequence` to card 8's `**Edit:**` list, and have the Intent state that their docs name both cases the clear is sent into: a box holding exactly the caller's own text, and a box holding exactly one stray-character candidate the guard reading found.

### [LOW] The reading after a held post-clear reading is unstated

**Location:** `_lyx/plan/08-stray-character-clear.md`, Intent paragraphs on `strayClear` and the residue reading.

**Issue:** `awaitIdleSession` polls until its window closes, and card 8 gives each attempt one `strayClear`, so the clear fires once.
The card says the residue reading "applies only on the post-clear reading", and that operator activity at that reading holds the send.
A held reading does not end the attempt: `awaitIdleSession` polls again.
For those later polls, with the box still holding the recorded character and the clear spent, the card reads two ways.
"Anything else after the clear holds as a draft" makes the box a draft until `send_ready_timeout_s` runs out, which is #554's stall again, for the length of one attempt.
Reading the character as residue on every later poll would instead admit it beside a full guard reading on a poll that is not the post-clear one.
The Stray character rows pin only the post-clear poll, so either implementation passes them.

**Fix:** state in card 8 which reading a later poll of the same attempt takes once the post-clear reading was held, and add that row to `TestSend_WaitsForIdleSession`'s Stray character rows, for example operator activity at the post-clear poll, then a quiet operator at the next poll with the character still on screen.

### [LOW] The new recording hooks' bound is unstated

**Location:** `_lyx/plan/01-hook-recording-capture.md`, Intent paragraph on `buildSettings`.

**Issue:** card 1 installs a `PermissionRequest` hook on every run and a `SessionStart` hook for every source.
A `PermissionRequest` hook's output and exit code feed Claude's permission decision, and a `SessionStart` hook's standard output joins the session's context, as the existing compaction entry relies on.
The existing `recordingCommand` writes only to the events file, and `buildSettings`'s `recordingHook` takes an `alwaysSucceeds` flag that every hook but `Stop` sets.
The card names neither: it never says that the two new recordings print nothing and so decide no permission and add no context, nor which `alwaysSucceeds` value they take.
The rubric's attack-surface question asks a card adding a routing edge to state what it can let through and what bounds it.

**Fix:** state in card 1's Intent that both recordings use `recordingCommand` with `alwaysSucceeds` true, print nothing, and so neither answer a permission request nor add to the session's context; `TestBuildSettings_RecordingHooks`'s new rows assert the `; true` suffix as they do for the other recording hooks.

### [LOW] Two cross-package surfaces ride in their consumer cards

**Location:** `_lyx/plan/02-idle-fresh-fold.md` (Create `SessionStartSource` and its four constants) and `_lyx/plan/08-stray-character-clear.md` (Create `StrayInputReader` and `Claude.StrayInput`).

**Issue:** the rubric's granularity rule gives a symbol that is part of a module's public surface, or reused across packages, its own card even when one card is its only consumer.
`SessionStartSource` and its constants are exported from `shuttleengine` and produced by `claudeengine`'s `sessionStartSource`; card 2 bundles them with the fold state, the readiness change and two idle-state enumerations.
`StrayInputReader` is a new exported capability that `claudeengine` implements, and its candidate table is a new top-level test; card 8 bundles them with the guard change, the confirm path and the PATTERN background edit.
Each pair is an independently reviewable unit: the source mapping and the candidate judgment can each be judged correct without the fold or the guard.
This is one method gap, filed once.

**Fix:** split each surface into its own card placed directly before its consumer with `lyx loom insert-card`: one card creating `SessionStartSource`, its constants and `claudeengine#sessionStartSource` with `SessionSignal`'s type change and the source rows of `TestParseSessionSignals`; one creating `StrayInputReader` and `Claude.StrayInput` with the candidate table and the `test-budget.yaml` row.
Cards 2 and 8 keep the fold and the guard.

## Focus verdicts

- D7 and D8 amendments against cards 7 and 8: card 7 takes the events-file size first on a wait's first `sessionGone` call, before the registry read, drops the read-alive disjunct, and carries every Death criterion row, the amended one included. The fold's dead reading beside a live registry is already pinned by `TestSessionFold`'s "a process-ending session end outranks even an unproven liveness" row and `settle`'s first case. Card 8 makes the post-clear reading a full guard reading and names it the write's guard reading, and the PATTERN.md line stays unchanged. Neither card adds scope the record does not license; card 9's notice change is D8's last clause.
- Card 8 attack surface: the Intent states what the clear can wipe (the character, plus anything typed between the reading and the clear) and its bounds. Each refusal branch is reached through `TestSend_WaitsForIdleSession` or `TestRun_Send_ConfirmsSubmission` by an input no upstream check rejects: a two-character box fails `StrayInput` and reaches the quoted draft hold, and `+` beside the sent text reaches `confirmPasted`'s `strings.Contains` branch. F2 and F3 are the gaps found.
- Card order: no card after 6 reads `RecordsSessionStarts` or `sessionStartRecorded`, and card 7's startup row needs only card 2's `SESSIONSTART:` fake and its own liveness script, so skipping card 6 breaks nothing. Cards 2 and 3 read the captured events file only after card 1 creates it, and cards 4 and 5 do not read it.
- Verify: `go test ./...` and `go test -tags integration ./...` run every targeted package, `go vet -tags llm ./...` compiles card 1's scenario, and the section follows the template's order. Every test file the `Edit:` lists name exists, `claudeengine/settings_test.go` holding `TestPrepare_WritesArtifactsAndReturnsConsistentLaunch` included.
- Test economy: `TestSmokeTurnsHookCapture` covers gates, registry readings, permission, clear, compact, resume and kill, which `TestSmokeTurnsSessionHooks` does not, and D1 places it beside that test. The `StrayInput` table pins a new method no existing test covers. Granularity is F5.
- Re-sign arrows: `busyReading`, `Runner.sendSessionSequence` and `innerRunProducer.noticeDriverBusy` carry the arrow, and each caller is on the same card. `confirmPasted` and `awaitIdleSession` keep their signatures, since the new state rides on `sendContext` and inside the poll loop, so they take none. The `Edit:` cards name member glyphs for every Go symbol their Intent changes.
