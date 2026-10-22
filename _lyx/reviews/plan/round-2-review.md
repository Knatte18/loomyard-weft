---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: decision
    location: _lyx/plan/08-verified-tree-hold-pattern.md:34
    summary: Cards 8 and 9 depart from the declared PATTERN-pause-holds-prompts text and edit eight other entry lines, and the decision record carries no Added after Discussion entry for either
    fix: No plan edit can remove the departure, since the declared line is 502 characters and pattern.MaxEntryLineChars is 360, so the fixer defers this finding to the parent (ly:orch) with that reason. The parent appends an `Added after Discussion` entry to _lyx/discussion/decision-record.md that records two things. First, the PATTERN.md line of PATTERN-pause-holds-prompts is a shortened summary because the declared text is over the entry-line cap, with the declared text carried in full in pattern/PATTERN-pause-holds-prompts.md. Second, the eight trailing-clause cuts of card 8 (PATTERN-sparse-hub-base, -config-layers, -mutation-record, -friction-capture, -testkit, -hubforge-fixtures, -gogit-read-helper, -cwd-resolution), moved word for word into their background files, are licensed because the overview is past pattern.HeadroomBytes and its headroom lint refuses any growth. Once the entry exists, cards 8 and 9 and the overview's Shared Decisions bullet need no change.
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/plan/03-verifytree-pause-hold.md:38
    summary: Card 3 states the marker is removed only on a hold cancel, but a Verify that held at entry and then returns StatusDirty or StatusSkipped must remove the paused marker its held callback wrote, and no row asserts it
    fix: In card 3's Intent, after the sentence that ends "the caller's cancel during the hold.", add that every return after a hold, whether StatusDirty, StatusSkipped, a cancel or the start over's own result, removes the marker, as Verify's doc says it does whatever happened. Then add to the TestVerify_PauseHold rows that return StatusDirty after the resume (the entry-hold row and the stop-then-dirty row) and to the StatusSkipped row the assertion that ReadMarker reports no live marker once Verify has returned. The `defer os.Remove(p.Marker)` must be registered before the first held callback can write the marker.
---

### [MEDIUM] Cards 8 and 9 rely on size departures the decision record does not record

**Location:** `_lyx/plan/08-verified-tree-hold-pattern.md:34`, `_lyx/plan/09-shuttle-send-refuses-covered.md:84`.

**Issue:** The decision record's `## PATTERN changes` declares `PATTERN-pause-holds-prompts` with a text that, rendered as an entry line, is 502 characters, over `pattern.MaxEntryLineChars` (360), which `pattern.Check` and `pattern-entry-line-cap` refuse.
Card 9 therefore writes a 262-byte summary line and puts the declared text in full into `pattern/PATTERN-pause-holds-prompts.md`.
The overview is 15703 bytes at the merge base, past `pattern.HeadroomBytes`, and `pattern.Headroom` fails any head longer than both the base and that line.
Card 8 therefore cuts 313 bytes of trailing clauses from eight unrelated entries into their background files.
I checked the figures: the eight clauses sum to 313 bytes, each occurs once in `PATTERN.md`, the verified-tree line grows by 29, and card 9's line is 262 bytes, so the overview ends 21 bytes shorter than at the merge base and card 8 runs before card 9.
The arithmetic and the card order are sound.
The gap is that `_lyx/discussion/decision-record.md` carries no `Added after Discussion` entry for either departure.
The line departs from a listed PATTERN entry, which is binding by its declared text, and the eight cuts are scope the answer key does not license.
Round 1's decision finding on these two cards remains open for the same reason.

### [LOW] Card 3 leaves the paused marker's removal after a hold unstated and unpinned

**Location:** `_lyx/plan/03-verifytree-pause-hold.md:38`.

**Issue:** Card 3's Intent has `held` write the `paused` marker while `Wait` holds at entry, and says the marker is removed on a cancel during the hold.
It then restarts from the dirty-tree check, which can return `StatusDirty` or `StatusSkipped` before the running marker is written.
In today's `Verify` the marker is written and then removed by a `defer` registered after the write; nothing in the card says the first `held` write is covered by a removal on those returns.
A Verify process that stays alive, such as webster's or the loop driver, would then leave a live `paused` marker behind, which loom status shows as `verify held by a pause` and which card 4's readers read as no activity.
The dirty and skipped rows of `TestVerify_PauseHold` assert the status and that no command ran, not that no marker remains.

## Answers to the focus entries

- **Card 3, injectable intervals:** not a defect.
  `PauseHold` exposes `Poll` for Verify's `PastGrace` poll and `Sleep` for the waits between `WaitUncovered`'s record reads, which `Waiter.Sleep` carries, and `NewPauseHold` sets today's intervals as defaults.
  `verifyrun`'s ten-second wait delay is not waited out by any row, since the stopped commands trap SIGTERM and exit, and card 2's table injects a short delay through `run`.
- **Card 3, redo of dirty and skip checks:** not a defect.
  `Wait` sits after the dirty and skip checks and before the running marker, and both a `Wait` that held and a stop at the grace end start over from the dirty-tree check, with rows for dirty, skipped and passed outcomes after the resume.
  F2 concerns only the marker, not the checks.
- **Card 3, `held` wiring:** not a defect.
  `held` is called from `Waiter.Sleep`, not from `Notify`, and `WaitUncovered` in `internal/pauserecord/hold.go` calls `sleep` only after finding a covering entry, so its unreadable-record notice, which returns nil before any sleep, never calls `held`.
  The card says so and has a row for an undecodable record.
- **Cards 8 and 9, `Added after Discussion`:** no such entry exists in the decision record, so F1 stands.
- **Card 14, resend refused again:** not a defect.
  A second `PausedError` calls `ClaimSession` once more with that error's slug and reports `claimed` true when it records the session or finds it owned, as D2 requires, with a row for it and rows for the ended-pause and claim-failed answers.
- **Cards 2, 9, 10, 15 Intents:** all four state what the focus asks.
  Card 2 states the failed group signal as a fail-loud wiring guard that takes no reaching test.
  Card 9 states `SendConditioned`'s replays read coverage and that its caller reads coverage itself.
  Card 10 states the refused segment-color line is logged and dropped.
  Card 15 names the Layering sentence listing what `Delivery` does, and the interrupt and Pause paragraphs of `internal/creelengine/doc.go`.
  None of them lists a file target beside a member of that file or a package target beside its members, so `redundant-file-target` and `redundant-package-target` do not bind.
