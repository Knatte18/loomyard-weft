---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/05-compact-cycle.md:Bound paragraph
    summary: Card 5 says card 11's smoke test asserts a background completion survives, but the smoke test is card 12; card 11 is adopt-pid-reuse
  - id: F2
    severity: LOW
    class: scope
    location: _lyx/plan/10-stencils-no-watcher.md:Start stencil bullet
    summary: Start stencil is told to explain how context is cycled without assuming /clear, a change D14 does not list and the card does not tie to a decision
  - id: F3
    severity: LOW
    class: design
    location: _lyx/plan/07-notice-queue.md:notice.go QueueNotice and watcher delivery
    summary: Notices are typed through Send, but QueueNotice accepts any line and no card states or tests the one-line bound at the queue or delivery
  - id: F4
    severity: LOW
    class: decision
    location: _lyx/plan/00-overview.md
    summary: The decision record's landing obligation (amend the orch-owns-the-loop board entry at landing) is carried by no card and is not named in the overview
---

### [MEDIUM] Card 5 names the wrong card for the smoke assertion

**Location:** `_lyx/plan/05-compact-cycle.md`, the `Bound:` paragraph.
**Issue:** It reads "card 11's smoke test asserts a completion survives".
Card 11 is `adopt-pid-reuse`; the compact smoke test with the background-task completion check is card 12.
An implementer following the pointer lands on the wrong card, and D7's bound ("the compact smoke test asserts a completion arrives") looks uncarried from card 5's side.
**Fix:** Change "card 11's" to "card 12's".

### [LOW] Start-stencil cycling wording is unlicensed

**Location:** `_lyx/plan/10-stencils-no-watcher.md`, Start stencil bullet, second sentence.
**Issue:** D14 gives the start stencil the notice rule only.
The sentence "It also says how the context is cycled without assuming `/clear`" adds stencil content the record does not list, and the card does not say which decision licenses it.
**Fix:** Tie it to D5 and D6 in the Intent (compact is the default, so a clear-only description would be false), limited to correcting that assumption.

### [LOW] One-line bound on typed notices is not stated at the queue

**Location:** `_lyx/plan/07-notice-queue.md`, `QueueNotice` and the delivery paragraph.
**Issue:** The decision record constrains stencils typed through `Send` to one line, and D12 makes the notice one line.
Card 8 renders it that way, but `QueueNotice` is the module-neutral entry that any caller can use, and the watcher types whatever file it finds.
The card states no bound on a multi-line or `/`-leading line reaching the pane, and no test pins it.
**Fix:** State in the Intent that `QueueNotice` refuses a line containing a newline or beginning with `/` with an error, and add that case to the `notice_test.go` list.

### [LOW] Landing obligation has no carrier

**Location:** `_lyx/plan/00-overview.md`.
**Issue:** The decision record lists, as a constraint, that the `orch-owns-the-loop` board entry's "A watch command" section is covered by D12 and D13 and is amended at landing.
No card does this, as it is a landing step, but the plan does not say so and the obligation can be lost.
**Fix:** Add one sentence to the overview's introduction naming it as a landing obligation outside the cards.
