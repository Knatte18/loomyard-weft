---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lym/plan/15-creel-post-interrupt.md:57
    summary: Card 15's journal-closing test row cannot reach the path it names, since an append after Store.Close panics instead of returning an error
    fix: >-
      In card 15's Tests paragraph, replace the row "an interrupt function that closes the store's journal makes `interruptTarget`'s `interrupted` append fail after the key" with a failure that returns an error.
      Close leaves the journal writer's file nil, and `journalWriter.writeLine` then builds its error message with `w.file.Name()`, a nil dereference, so the append panics inside `Store.do` with the store mutex held.
      Name instead: an interrupt function that advances the fake clock past the next UTC midnight and removes the journal directory, so the `interrupted` append rolls the day file, `rotate` fails to open the new file, and `append` returns an error.
      Keep the row's assertions: the error says the turn was interrupted and the post was not stored, and names the post without `--interrupt`.
      The second row, whose interrupt function calls `SetStrandsError`, reaches the re-read failure and stays as written.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lym/plan/18-creel-hooks-llm-smoke.md:22
    summary: The bell assertions of the idle-wake and interrupt-and-bell scenarios read the pane, where the full bell line wraps and scrolls out, so they can fail or pass vacuously with no code defect
    fix: >-
      In card 18, state the bell assertions through the events file, not the pane.
      The typed bell starts a turn, so its `UserPromptSubmit` payload line in `events.jsonl` holds the bell as its prompt, while a ring-woken turn records none.
      Write "the bell line is never typed" in the idle-wake scenario as: no `UserPromptSubmit` payload whose prompt equals `creelengine.BellLine`.
      Write "typed exactly once, and no second bell for that set" in the interrupt-and-bell scenario as: exactly one such payload, counted after the message has shown.
      Keep the pane polling for the message text, which is short, and add `plan:internal/creelengine#BellLine` under **Uses:**.
      The fix passes `lym loom validate-plan --overlay`, alone and combined with F3's.
  - id: F3
    severity: LOW
    class: design
    location: _lym/plan/15-creel-post-interrupt.md:38
    summary: Card 15 has post wrap each of interruptTarget's three refusals with its own way forward, but the refusals are plain errors a caller cannot tell apart and no target edits them
    fix: >-
      In card 15, move `internal/creelengine#Store.interruptTarget` from **Uses:** to **Edit:** and add `internal/creelengine#holdRefusal` and `internal/creelengine#unreadablePauseRefusal` under **Edit:**.
      Add one sentence to the Intent: `interruptTarget` returns its hold, lifted-pause and unreadable-pause refusals as a typed error carrying the refusal's kind, which `Interrupt` prints as it does today and `PostInterrupt` matches with `errors.As` to append its own way forward.
      Also say what `PostInterrupt` answers for an error `interruptTarget` returns with false that is none of the three, a failed key: it is returned as `interruptTarget` words it, with nothing stored.
      The Edit-list change passes `lym loom validate-plan --overlay`, alone and combined with F2's.
  - id: F4
    severity: LOW
    class: scope
    location: _lym/plan/07-shuttle-idle-no-turn.md:36
    summary: Card 7's guarded rows pin only the draft and running-turn cases, so the guarded branch's too-short and no-input-box conditions have no row that reaches them
    fix: >-
      In card 7's paragraph on the `guarded` setting, add two rows that run on `guardedReadinessEngine` under an unknown reading: a too-short pane (`tooShort` set) reads `NoTurn` false, and a pane showing no input box reads `NoTurn` false, each with `Idle` false and a zero `TurnEndedAt`.
      Without them the "too-short pane under the pane probe" row can run on the unguarded engine, where `NoTurn` equals the pane's idle answer, so a guarded branch that drops its `readBox` found or not-too-short condition passes every row.
      Say the too-short row of the New rows paragraph runs with `guarded` set.
  - id: F5
    severity: NIT
    class: consistency
    location: _lym/plan/08-creel-delivery-seam-fields.md:37
    summary: Card 8's Bound sentence says hand-added strands reach the unregistered-provider branch, but a hand-added strand has no provider and takes the empty-provider branch
    fix: >-
      In card 8's Bound sentence, replace "so the unregistered-provider branch is reached in tests by providers such as `codex` and `script`, and in production by hand-added strands" with "so the unregistered-provider branch is reached only in tests, by providers such as `codex` and `script`, and the empty-provider branch is what hand-added strands reach in production".
---

### [MEDIUM] Card 15's journal-closing test row panics instead of reaching its path

**Location:** `_lym/plan/15-creel-post-interrupt.md:57`

**Issue:** The focus asks what `post --interrupt` answers when `interruptTarget` returns true together with an error, and whether each store-failure row reaches its path.
The answer is stated: the error is treated as a store failure after the key and reads "the turn of `<receiver>` was interrupted, but the post was not stored", which agrees with D7.
The first planned row does not reach it.
`Store.Close` calls `journalWriter.close`, which sets the writer's file to nil.
The `interrupted` append then calls `writeLine`, where `w.file.Write` returns `ErrInvalid` and the error message is built with `w.file.Name()`.
`Name` on a nil `*os.File` dereferences nil; I ran that call in a scratch program and it panicked.
The panic fires inside `Store.do`, which holds the mutex without a deferred unlock, so the row would crash the test binary and never see the planned error.
No card in the plan changes `writeLine`.
The second row, whose interrupt function calls `SetStrandsError`, reaches the re-read failure.
Its `strandTarget` read happens before the key, so only the later re-read fails, as the row says.

### [MEDIUM] Card 18's bell assertions read the pane and can fail with no defect

**Location:** `_lym/plan/18-creel-hooks-llm-smoke.md:22`

**Issue:** The focus asks whether the interrupt-and-bell assertions can fail with no code defect, given what Claude Code does with an idle waiter across a later turn.
The waiter premise holds.
The interrupted turn is the session's first and a user interrupt fires no `Stop`, so no waiter exists and each scenario launches its own session, so none carries over; the undocumented waiter lifetime does not touch these assertions.
The assertions fail on another ground: how they observe the bell.
The existing smoke tests poll `reed.CapturePane`, which runs `capture-pane -p` with no `-J` and no scrollback.
The full bell line is about 135 characters, so in a pane narrower than that it wraps, with Claude Code indenting the continuation, and a whole-line match finds nothing.
A "typed exactly once" check can then read zero, and the idle-wake "never typed" check passes vacuously because the line has scrolled out or wrapped.
The model can also quote the line, since the ring delivers the identical text as a system reminder.
The events file is already this test's evidence for the Stop record, and a typed bell starts a turn whose `UserPromptSubmit` payload carries it as the prompt, while a ring-woken turn records none.

### [LOW] Card 15 wraps three refusals it cannot tell apart

**Location:** `_lym/plan/15-creel-post-interrupt.md:38`

**Issue:** Card 15 says `post` wraps a hold refusal, a lifted-pause refusal and an unreadable-pause refusal each with its own way forward.
In the tree, `holdRefusal` and the lifted-pause error are plain `fmt.Errorf` values, and `unreadablePauseRefusal` wraps an arbitrary cause.
No sentinel or type separates them, and card 15 lists `interruptTarget` under **Uses:**, so no target allows the change that would.
An implementer must either match message text or edit symbols the card does not name.
A key that the delivery refuses, the `default` case of `interruptTarget`, returns a raw error that the card says nothing about.

### [LOW] Card 7 leaves the guarded branch's box conditions unpinned

**Location:** `_lym/plan/07-shuttle-idle-no-turn.md:36`

**Issue:** The focus asks whether some row reaches the pane-probe `NoTurn` branch for an engine implementing `InterruptGuard` under an unknown reading, with and without a running turn.
Both do: a draft with no running turn reads `NoTurn` true and the same pane showing a running turn reads false, on `guardedReadinessEngine` with an unproven session.
The branch has two more conditions in the Intent, that `readBox` found the input box and that the pane is not too short.
The rows that pin those, a too-short pane under the pane probe and a pane with no box, do not say they run on the guarded engine, and on the unguarded one `NoTurn` is the pane's idle answer, so they never exercise those conditions.

### [NIT] Card 8's Bound sentence names the wrong branch for hand-added strands

**Location:** `_lym/plan/08-creel-delivery-seam-fields.md:37`

**Issue:** The treatment itself agrees across the overview's Shared Decisions, card 8's Intent and `TestHubDelivery` rows, card 9's Intent and `TestSmokeCreelClaudeStrand` change, card 15's help text and card 17's sandbox note: an empty provider is a pane-mail strand.
Only the Bound sentence misplaces the branch, as the fix describes.
The mislabel can send an implementer to look for a provider tag on hand-added strands that `lym reed add` never writes.

## Rejected

- Focus 1, empty-provider delivery: not a defect.
  A strand with an empty `Provider` is a pane-mail strand in every card that states a treatment, and `TestSmokeCreelClaudeStrand` stays unchanged because its hand-added strand keeps the typed path.
  A `lym shuttle open` session records `Provider` as its engine name, so it is a Claude strand without the mark and gets neither path, as D5 and D6 say.
- Focus 3, whether a row reaches the guarded pane-probe branch with and without a running turn: settled as above, with the remaining gap filed as the card 7 finding.
- Focus 4, the waiter premise of the interrupt scenario: not a defect, and the observation method is filed as the card 18 finding.
