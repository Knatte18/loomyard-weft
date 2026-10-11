---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D7 (pack fragment) and D4 (unresolved heading)
    summary: The pack fragment tells the fork to locate every unresolved entry with quarry or grep, but D4 also lists plan handles there, which name members not yet created, and gives them no stated reason.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D2
    summary: D2 defines the logged handles as BindHandles counts them, but the card-not-done, batch-not-done and language-none paths never get a BindHandles outcome, and D2 names no source websterengine can enumerate a card's owned handles from.
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D2 (the only-passes-that-log-no-bind-lines sentence)
    summary: The closed list of passes that log no bind lines omits the Index lookup error postBatchChecks returns before DoneChecks runs.
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/support-log.md Interview item 6 (stale _lyx path bullet)
    summary: The support log records the stale webster-prefix-fork.md banner path as fixed in Notes for the plan writer, but the Notes no longer carry it and Scope does not include it.
---

### [MEDIUM] Unresolved `plan:` handles get a locate instruction that cannot apply

**Location:** `_lyx/discussion/decision-record.md` D7, the pack fragment's content; D4, the unresolved heading.

**Issue:** D4 lists "every `plan:`-shaped ref on the batch's cards, under any label" under the unresolved heading "with its quarry status or reason".
D7's pack fragment then says "an unresolved entry is located with `lyx quarry resolve` or a grep, since a member glyph names a package, not a file".
For an unresolved member glyph this is right, and it is consistent with the ban, which D7 bounds to resolved entries; the focus question's first half is settled as not a defect.
For a `plan:` handle it is wrong: today's second locate sentence (`webster-body-implementer.md:35`, "A `plan:` handle names a member the card creates and does not resolve yet") is what tells the fork there is nothing to locate, and D7 moves that sentence into the quarry fragment only.
The pack fragment's enumerated content drops it, so with the switch on the fork is told to quarry or grep for a member that does not exist yet.
D4 also names no reason for a `plan:` entry: it is never a quarry `not_found` or `ambiguous` answer, and the acceptance criterion "a `plan:` handle … appear[s] under the unresolved heading" pins its presence, not what the fork is told about it.
On the focus question's second half: the fragment's "for each member glyph on the cards" agrees with D4's exclusion of self glyphs and paths, but not with D4's inclusion of `plan:` handles, which the fragment never mentions.

**Fix:** In D7, have the pack fragment carry the `plan:` handle meaning (a handle names a member a card creates, so there is nothing to locate), scope the locate instruction to unresolved member glyphs, and in D4 state the fixed reason a `plan:` entry carries.
Extend the D7 acceptance criterion so the `claims_test.go` row pins that sentence.

### [MEDIUM] D2 has no stated source for a card's owned handles on the paths that skip `BindHandles`

**Location:** `_lyx/discussion/decision-record.md` D2.

**Issue:** D2 counts "`plan:` handle[s] owned by the batch's cards (Create declarations and handle-shaped Rename new sides, as `BindHandles` counts them)" and has `BindHandles` return the per-handle outcome.
Three of the six reasons arise where no such outcome exists.
`card-not-done` and `batch-not-done`: `postBatchChecks` (`internal/websterengine/recordbatch.go:779`) returns the not-done refusal before `BindHandles` is called.
`language-none`: `BindHandles` (`internal/planglyph/handle.go:412`) returns `nil, nil` before it enumerates any handle, so "as `BindHandles` counts them" counts nothing.
The ownership rule lives in `cardOwnHandles`, unexported in `internal/planglyph`, which `internal/websterengine` cannot import (D5 keeps it free of quarry).
So websterengine, which D2 makes the writer of every line, has no named way to list a card's handles on those paths; a plan writer must either duplicate the ownership rule in websterengine or invent a new boundary, and D2 decides neither.
This is a missing module-boundary decision, not a call-site enumeration.

**Fix:** Decide in D2 where the per-card owned-handle list comes from on every path, for example an ownership function declared once by `internal/planparser` (per `PATTERN-ref-shape-registry`) and used by both `BindHandles` and websterengine, or a `planindex` method; and define the logged handle set by that rule instead of "as `BindHandles` counts them".

### [LOW] D2's closed list of no-line passes omits the `Index` lookup error

**Location:** `_lyx/discussion/decision-record.md` D2, "The only passes that log no bind lines are those whose `DoneChecks` call errors, …".

**Issue:** `postBatchChecks` first calls `in.Geom.index()` and returns its error (`internal/websterengine/recordbatch.go:764-767`), before `DoneChecks`.
That pass logs no bind lines, but D2 says "the only passes" are the three named, and the acceptance criterion lists the same three.
On the focus question: otherwise D2's entry and exit conditions agree with the code — a not-done card and an unmatched handle both end in refusals after lines are logged, and `ErrQuarryUnavailable` is tolerated and maps to `delta-unavailable`.

**Fix:** Add the `Index` lookup error to D2's list of passes that fail before logging, or phrase the list as "any error before or from `DoneChecks`, …".

### [LOW] Support log claims a fix the decision record no longer holds

**Location:** `_lyx/discussion/support-log.md`, Interview item 6, "a stale `_lyx` path in the `webster-prefix-fork.md` banner — fixed: Notes for the plan writer".

**Issue:** The banner comment in `contracts/stencils/webster/webster-prefix-fork.md:2` still says `_lyx/webster/prompts/`, while D6 places prompts under `.lyx/webster/prompts/`.
The decision record's Notes for the plan writer carry no such note, and Scope does not list the banner.
Round 1's fixer removed the note as out of scope, which is a valid disposition; the support log's line was not updated with it.
On the focus question about Notes: each remaining note is a head-start pointer and none states a requirement outside Scope, so the Notes themselves are not a defect; the support log's disposition is what disagrees.
Every Constraints bullet's cited source does list what it constrains: `PATTERN-config-strictness` names `websterengine` among degrading callers, and the `planindex` package doc states its standard-library-plus-`planparser` import set.

**Fix:** Change the support-log disposition to say the note was dropped as outside this task's scope.
