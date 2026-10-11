---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D7
    summary: The pack fragment bans quarry and grep, but its fallback for an unresolved member glyph, reading the card's named files, names no file, since a member glyph's unit is a package
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md D2
    summary: D2's entry and exit conditions, read literally, exclude the passes that produce four of its five reasons
  - id: F3
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md D2
    summary: D2 promises exactly one line per handle, but a language none plan may carry handles that BindHandles never binds, and no reason in the closed set covers that
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md Acceptance criteria
    summary: D2's error-path rule, no bind lines when the verb fails with an error, is pinned by no acceptance criterion
  - id: F5
    severity: LOW
    class: scope
    location: _lyx/discussion/decision-record.md Notes for the plan writer
    summary: The stale _lyx path in the webster-prefix-fork.md banner sits in Notes without being in Scope or out of it
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md Constraints
    summary: The planindex import constraint cites PATTERN-leaf-packages, whose background does not list planindex
---

### [MEDIUM] Unresolved member glyphs have no locate path under D7's ban

**Location:** `_lyx/discussion/decision-record.md` D7, with D4 and D5.

**Issue:** The pack fragment tells the fork not to call `lyx quarry resolve` and not to grep for a listed member, and says "an unresolved entry is located by reading the card's named files".
Unresolved entries are listed too, under D4's unresolved heading, so the grep ban reads as covering them.
A Go member glyph's unit is a package directory (`internal/x#Foo`; `loom-plan-spec.md` shape rule 2), so a card whose refs are all member glyphs names no file.
Under D5's whole-call resolve error, every member glyph lands unresolved, so this case covers the whole batch.
The fork is then left with no permitted way to locate the member except reading every file in the package.
The stale-span case is bounded: the listed signature plus the listed file suffice.
The fragment also says the pack lists "each card glyph", while D4 leaves out self glyphs and paths.

**Fix:** In D7, scope the quarry and grep ban to resolved entries, and state the locate path for an unresolved entry: `lyx quarry resolve` or a grep is allowed for it.
Say "each member glyph" instead of "each card glyph".
Add the unresolved-entry instruction to the pack fragment's acceptance criterion.

### [MEDIUM] D2's conditions contradict its own reasons

**Location:** `_lyx/discussion/decision-record.md` D2.

**Issue:** D2 opens with "every terminal pass that gets past its done-check and delta calls without an error" logs the lines, and closes with "a pass that ends in an error … logs no bind lines".
In `postBatchChecks` (`internal/websterengine/recordbatch.go`), a not-done card returns `cardNotDoneError`, an unmatched handle returns `ErrCardNotDone`, and the `ErrQuarryUnavailable` delta call returns an error, which the pass tolerates.
So `card-not-done`, `batch-not-done`, `no-delta-match`, `sibling-unmatched` and `delta-unavailable` all come from passes that either do not get past a call "without an error" or that end in an error.
Only the parenthetical list of errors shows the intended reading.
A plan writer who follows the general phrasing logs lines only for fully bound batches.

**Fix:** Restate the rule in terms of outcomes: the lines are logged on every pass except one whose `DoneChecks` call errors, whose `Delta` call errors with anything other than `ErrQuarryUnavailable`, or whose `BindHandles` call errors.
The verb's not-done refusal is not one of these exceptions.

### [MEDIUM] `language: none` plans with handles fall outside D2's closed set

**Location:** `_lyx/discussion/decision-record.md` D2.

**Issue:** `loom-plan-spec.md` says the handle checks "run under every `language:`, including `\"none\"`", so a `language: none` plan may declare `plan:` handles.
`BindHandles` returns `nil, nil` right away when `plan.GlyphLanguage()` reports no language (`internal/planglyph/handle.go`), so those handles are never bound and get no per-handle outcome.
D2 still promises "exactly one line per `plan:` handle" with one reason from a closed set, and none of the five reasons fits.
D5 and D9 handle `language: none` for the pack, but D2 does not handle it for the trace.

**Fix:** Decide in D2: either a `language: none` plan logs no bind lines, or a sixth reason (for example `language-none`) joins the closed set.
Add the matching scenario to the D2 acceptance criterion.

### [LOW] The error-path rule has no criterion

**Location:** `_lyx/discussion/decision-record.md` Acceptance criteria, second bullet.

**Issue:** The support log records the error-path rule as a fixed lens finding (Interview 6, "error paths log nothing").
It is in D2, but the D2 criterion's scenario list does not include a pass that ends in a `DoneChecks`, `Delta` or `BindHandles` error and logs no bind lines.
The other parts of the focus check hold: each of the five reasons and the `recover-batch` sharing has a scenario.
D2's reasons do not contradict D4: D2 counts handles a card owns and binds, while D4 lists every `plan:` ref in the pack, and the two lists serve different consumers.

**Fix:** Add a scenario to the D2 criterion: a pass that ends in one of D2's named errors logs no bind lines.

### [LOW] The stale banner path is a stray note

**Location:** `_lyx/discussion/decision-record.md` Notes for the plan writer, third bullet.

**Issue:** The `webster-prefix-fork.md` banner does say `_lyx/webster/prompts/`.
No Decision edits that stencil: D7 changes `webster-body-implementer.md` and adds new stencils, and the pack-section marker has to render as nothing in the recovery prompt, so it sits in the shared body.
That leaves the note neither a requirement nor a head start.
`Plan-Write` treats Notes as optional, so the fix happens only by chance.

**Fix:** Pick one.
If the task fixes the banner, move it into Scope In next to the docs bullet.
Otherwise, remove the note from the decision record.

### [LOW] Wrong PATTERN cited for the `planindex` import set

**Location:** `_lyx/discussion/decision-record.md` Constraints, sixth bullet.

**Issue:** `pattern/PATTERN-leaf-packages.md` lists the closed-import leaf packages, and `internal/planindex` is not among them.
The `planindex` import set (standard library plus `planparser`) comes from that package's own `doc.go`.
The bullet's requirement is right, but naming `PATTERN-leaf-packages` points the plan writer at a background file that says nothing about `planindex`.
On the focus question: D2's outcome type and D5's new `Index` method's types both fall under this constraint, so they agree.
D5 does not name the owning package of the pack entry types, but the constraint already implies `planindex`, and naming an interface shape is outside what the writer stencil gathers.

**Fix:** Cite the `planindex` package doc alone, and drop `PATTERN-leaf-packages` from this bullet.

## Focus verdicts without a finding

- D3 and D5 bounds: the degraded and mixed states each name what they let through and what bounds it.
  D5 lists every member as unresolved and logs a Warn line, after `ValidateDispatch` has already validated the batch.
  D3 lets a flip apply from the next `begin-batch`, and D9's per-batch line records it.
  Not a defect.
- Support-log-only requirements: the operator's "still reads the code" rule is in D7's fragment text, its rationale and a criterion.
  The same-binary comparison is in D3, the no-cap pick is under Auto-mode assumptions, and the `webster-spec.md` premise is in Scope.
  None exists only in the support log.
- Leanness: Rationale is required by the writer stencil, and the Auto-mode bullets are one-line references.
  I found no deliberation narrative or out-of-scope content in the decision record that needs moving, beyond F5.
