---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: _lyx/plan/11-shedrecipe-multillm-entry.md:11
    summary: Card 11 registers MultiLLM but does not target TestNames ShipsExpectedEntries in registry_test.go, which pins the exact registry list and fails the card gate
  - id: F2
    severity: MEDIUM
    class: scope
    location: _lyx/plan/06-seatengine-run.md:43
    summary: Card 6 queues a chair notice for every advisor end, including Done, where D7 says an advisor reaching done draws nothing
  - id: F3
    severity: MEDIUM
    class: scope
    location: _lyx/plan/06-seatengine-run.md:42
    summary: Card 6 leaves the live strand of a seat that fails checkName unstopped, so a misnamed chair keeps running and a later probe can attach it
  - id: F4
    severity: MEDIUM
    class: scope
    location: _lyx/plan/03-seat-directive-stencils.md:13
    summary: Cards 3 and 5 open the advisor-to-chair message edge and the chair's delegate authority over advisors with no stated bound in their Intent
  - id: F5
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/05-seatengine-prompts.md:31
    summary: Card 5 plans a test that a seat stencil missing a reserved marker is refused by the fill, but FillWith refuses only unfilled declared markers and no card builds that check
  - id: F6
    severity: LOW
    class: scope
    location: _lyx/plan/03-seat-directive-stencils.md:15
    summary: The chair block as planned carries no channel wording (SendMessage by strand name), though D3 puts that wording in both blocks
  - id: F7
    severity: LOW
    class: scope
    location: _lyx/plan/06-seatengine-run.md:46
    summary: Card 6 does not say what join does when the chair's Wait returns an error rather than a terminal result
  - id: F8
    severity: LOW
    class: consistency
    location: _lyx/plan/11-shedrecipe-multillm-entry.md:39
    summary: Card 11's construction table re-tests the unknown row key, which TestEveryEntry_RejectsUnknownConfigKey already covers once minimalEntries gains the MultiLLM row
  - id: F9
    severity: LOW
    class: consistency
    location: _lyx/plan/12-multillm-fixture-recipe.md:20
    summary: Card 12's nested-include assertion repeats the refusal card 4's Validate table and card 11's construction table already pin
  - id: F10
    severity: LOW
    class: consistency
    location: _lyx/plan/05-seatengine-prompts.md:22
    summary: Card 5 Uses the embedded SeatDirectiveChair and SeatDirectiveAdvisor vars, though seatengine reads blocks from the told directory and its test seeds them via stencilkit
  - id: F11
    severity: NIT
    class: consistency
    location: _lyx/plan/05-seatengine-prompts.md:14
    summary: Card 5 Edits Table.Validate only to re-read a list card 4 already declares; ReservedMarkers belongs in card 4
---

### [BLOCKING] Card 11 misses the registry-name pin

**Location:** `_lyx/plan/11-shedrecipe-multillm-entry.md:11`

**Issue:** `internal/shedrecipe/registry_test.go`'s `TestNames` subtest `ShipsExpectedEntries` compares `Names()` against a fixed list of every registered engine, and fails on any length or order difference.
Card 11 adds `MultiLLM` to `registry` but its `**Edit:**` list names `minimalEntries` and `coverageGuardAllowedUnreachableEngines` only, not `TestNames`.
The card gate runs the untagged suite, so card 11 fails its own gate, or its implementer edits a file the card does not license.

**Fix:** Add `internal/shedrecipe#TestNames` (or `internal/shedrecipe/registry_test.go`) to card 11's `**Edit:**` list, and say in the Intent that `MultiLLM` joins the `ShipsExpectedEntries` list in sorted position.

### [MEDIUM] Card 6 notifies the chair when an advisor reaches Done

**Location:** `_lyx/plan/06-seatengine-run.md:43`

**Issue:** The Intent says each started advisor's `Wait` is joined and "its end is queued on the notice sender as an `advisorNotice` line", with no exception for Done.
With `KeepPane`, an advisor's `Wait` settles Done once its outputs exist, so every advisor that finishes normally types a line into the chair.
D7 closes the rules with "Advisor reaches done: nothing happens; it stays reachable (D5) and the chair reads the file", and the acceptance list draws a notice only for an advisor that dies or times out.
The resume path (card 7) correctly words a notice for a finished advisor, but only because the probe removed its seat; that is not the in-call case.

**Fix:** State in card 6 that a Done advisor's end is recorded on its `SeatResult` and queues nothing, that only died and timed-out ends queue an `advisorNotice` line, and add a row to `engine_test.go`'s Done-advisor case asserting no send.

### [MEDIUM] Card 6 leaves a misnamed seat's strand running

**Location:** `_lyx/plan/06-seatengine-run.md:42`

**Issue:** `startSeat` runs `checkName` after `StartGated` has returned a live handle; a mismatch is a start error, but the card never says the started strand is stopped.
For an advisor, the strand keeps running under a numbered name while the chair's prompt lists it in `failed_advisors`.
For the chair, line 44 stops "every started advisor" and returns, leaving the chair's own strand live with a running record over the step's output files; the next call's chair probe (card 7) finds that record attachable and resumes it, so the advisors address a chair name that the holder answers.
D4 makes the mismatch a start error so a seat is never silently misaddressed; leaving the strand running reintroduces that.

**Fix:** In card 6, stop the handle through `stopSeat` when `checkName` fails, before recording the start error, and return `ErrSeatNotStopped` with the way forward when that stop fails; add the chair and advisor rows to `engine_test.go`'s formed-name case asserting the stop.

### [MEDIUM] No bound stated for the seat channel and the chair's authority

**Location:** `_lyx/plan/03-seat-directive-stencils.md:13`, `_lyx/plan/05-seatengine-prompts.md:28`

**Issue:** Card 3 introduces the advisor-to-chair message route (the advisor block's `SendMessage` to the chair), and card 5 renders an advisor's `parent_directive` with the chair's strand name, which makes a chair message an operator-grade instruction to the advisor.
Both are new routing edges.
D6 states what each lets through and what bounds it: the message-target rule is prompt-enforced only, an advisor's message carries no delegate authority, and a relayed chair instruction is bounded by the advisor's write confinement, its settings' denies and its never being interactive.
Neither card's `**Intent:**` states any of that.

**Fix:** Add one sentence to card 3's Intent stating that the advisor-to-chair rule is prompt-enforced, that a disobeying advisor can reach any name `ListAgents` shows, and that no advisor prompt names another session and advisors are never interactive or noisy.
Add one sentence to card 5's Intent stating that naming the chair as an advisor's parent lets a chair message through as an instruction, bounded by the advisor block's own-output write rule, its settings' denies and `Interactive` false.

### [MEDIUM] Card 5 plans a refusal that nothing builds

**Location:** `_lyx/plan/05-seatengine-prompts.md:31`

**Issue:** `prompt_test.go` is to cover "a seat stencil missing a reserved marker refused by the fill".
`stencil.FillOptional` (and so card 2's `FillWith`) refuses a marker the template declares but the values leave empty; it never refuses a value whose marker the template lacks.
A seat stencil that omits `{{.parent_directive}}` therefore fills without error, and no card plans a check that a seat stencil renders the reserved markers.
The implementer must either write a test that cannot pass or invent an unplanned check.

**Fix:** Reword the row to the refusal the fill makes: a seat stencil declaring a marker absent from its value map is refused with that marker named.
If the plan wants seat stencils held to rendering `parent_directive` and `edit_directive`, add that check to `Table.Validate` in card 4 with its own row, as the Constraints' PATTERN-parent-directive line asks of consumer stencils.

### [LOW] The chair block lacks the channel wording

**Location:** `_lyx/plan/03-seat-directive-stencils.md:15`

**Issue:** The advisor block names the mechanism ("by `SendMessage` to its strand name"); the chair block says the advisors "answer its questions over the channel" without saying how the chair asks.
D3 says both blocks carry the seat-to-seat channel wording, and that wording lives nowhere else, so the chair's prompt is left without it.

**Fix:** Add to the chair block's planned content that it asks an advisor by `SendMessage` to the advisor's strand name from `advisor_names`.

### [LOW] Chair Wait error is unspecified

**Location:** `_lyx/plan/06-seatengine-run.md:46`

**Issue:** `join` waits on the chair, then stops advisors and maps "the chair's outcome".
The card never states the case where the chair's `Wait` returns an error, which carries no terminal outcome and may leave the chair running.
D5 stops advisors on a terminal result only; the burler precedent the card cites stops a half whose wait errored, since it may still be running.

**Fix:** State the rule in card 6: on a chair `Wait` error, stop the chair and every live advisor and return the error, as burler's `decideRound` does, or leave all seats for the next probe; add the row to `engine_test.go`.

### [LOW] Card 11 re-tests the unknown row key

**Location:** `_lyx/plan/11-shedrecipe-multillm-entry.md:39`

**Issue:** `internal/shedrecipe/conformance_test.go`'s `TestEveryEntry_RejectsUnknownConfigKey` constructs every registry entry from its `minimalEntries` row and asserts a `bogus_key` is rejected.
Card 11 adds the `MultiLLM` row to `minimalEntries`, so that test covers the unknown row key; the "unknown row key" row in the new construction table repeats it (PATTERN-test-economy).

**Fix:** Drop the unknown-row-key row from `entries_multillm_test.go`'s construction table and keep the unknown-seat-key row, which the conformance test does not reach.

### [LOW] Card 12 repeats the nested-include refusal

**Location:** `_lyx/plan/12-multillm-fixture-recipe.md:20`

**Issue:** Card 4's `Validate` table covers a nested include, and card 11's construction table covers "each `Table.Validate` refusal reaching the caller" through the entry.
Card 12's "a seat stencil with a nested include fails the build" asserts the same refusal a third time through `shedbuild`.

**Fix:** Drop the nested-include assertion from `multillm_fixture_test.go`; the fixture test's job (D13) is the build and the Done run.

### [LOW] Card 5 names the embedded seat-block vars as uses

**Location:** `_lyx/plan/05-seatengine-prompts.md:22`

**Issue:** `Uses:` lists `plan:contracts/stencils#SeatDirectiveChair` and `#SeatDirectiveAdvisor`.
The Intent reads every block from the told directory through `stencilstore.Read`, as PATTERN-stencil-ownership requires, and the test's seeded directory comes from `stencilkit.Seed`, which seeds the registry.
Neither the code nor the test reads the vars, and listing them invites a production import of embedded bytes, outside D1's import set.

**Fix:** Replace the two entries with `internal/testkit/stencilkit#Seed`.

### [NIT] ReservedMarkers is split across cards 4 and 5

**Location:** `_lyx/plan/05-seatengine-prompts.md:14`

**Issue:** Card 4 declares the reserved-name list in `table.go` and has `Validate` read it; card 5 wraps it in `ReservedMarkers` and Edits `Validate` so it "now reads that list through the same declaration", which it already does.
The Shared Decision `reserved-markers-declared-once` reads more simply with the list and its accessor born together.

**Fix:** Move `ReservedMarkers` to card 4's `**Create:**` and drop `Table.Validate` from card 5's `**Edit:**`.

## Fidelity check

Every Decision D1–D14 and every Constraint in the decision record is carried by some card: D1 by cards 4–9, D2 by 4 and 11, D3 by 2, 3 and 5, D4 by 1, 4, 5 and 6, D5 by 6 and 9, D6 by 3 and 5, D7 by 1 and 6, D8 by 7 and 9, D9 by card 6's start order and `Spec`'s default below-parent display, D10 by 1 and 5, D11 by 9, D12 by 10 and 11, D13 by 4–13, D14 by 1, 4, 6, 7, 9, 11 and 14.
`Env.Models` (card 10) is licensed by D2's "the registry the `Env` carries"; `Env.Landing.Registry` is filled only in `run.go`, not `wire`, so it cannot serve.
`Run.StrandName` (card 1) is licensed by D4's name comparison.
No card adds scope the record does not license; F2 is a departure from D7, not added scope.

Card order holds: every `Uses` and `Edit` reaches a symbol an earlier card creates (card 4's `IncludeNames` from card 2, card 5's `FillWith` and seat blocks from cards 2 and 3, card 8's seatengine types from cards 6 and 7, card 9's fake from card 8); card 8 satisfies card 9's `SeatRunner` structurally and asserts nothing against it.

Edit-card glyphs: cards 1, 2, 10 and 11 list member glyphs, none changes a signature, and `doc.go` targets carry no symbol.

Verify: the `## verify:` section is in the template's order and covers every targeted package through `./...`, with `integration` run, `tmux` and `llm` compiled; no card targets a nested module.
