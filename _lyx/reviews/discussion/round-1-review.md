---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md D2 (apply order and partial failure)
    summary: A mid-apply I/O failure joins the loom raw-I/O row, whose contract is nothing mutated and re-run, but the plan is left mutated and the verbs' own consistency precondition refuses the re-run, so the partial state has no way forward
  - id: F2
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md D2 (insert-card slug refusal)
    summary: The plan format has no slug rule beyond non-whitespace, so the slug bound cites a rule that does not exist and a slug holding / or .. lets insert-card write outside the plan directory
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D2 (begun-card bound)
    summary: The verbs read the run record and then apply without holding webster's state-mutation lease, so a begin-batch between the read and the apply can record a batch whose card the verb then renumbers
  - id: F4
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md D1 and D2 (--intent flag)
    summary: The --intent value is written into the Card Index unbounded; an empty or multi-line intent breaks the index line the verb itself writes
  - id: F5
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md Open risks (first bullet); support-log.md Q9
    summary: The rebaseline question is answered by the code today and is left open as a plan-time check, which the writer stencil forbids as a deferral
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D2 and D3 (run-record reading)
    summary: D2 and D3 say the begun set is read the way loomshed.ValidatePlan reads the run record, but ValidatePlan reads only the terminal-done set; only the Notes say begun differs
  - id: F7
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md D5 (writer rule scope)
    summary: The reachability rule covers every prescribed error return, which takes in propagated raw I/O errors that refusal-spec groups in one transient row per section
  - id: F8
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md Open risks (second bullet)
    summary: The residual-risk note about distant upstream rejections carries no requirement for the plan writer and belongs in support-log.md
---

### [BLOCKING] Partial apply leaves a plan nothing can repair

**Location:** `_lyx/discussion/decision-record.md` D2, the paragraph starting "The verbs compute every new file name and content in memory first".

**Issue:**
The move orders are correct: descending for insert and ascending for remove (after deleting card `N`) never let two files claim one number, and writing the Card Index last is sound.
The failure path is the defect.
D2 says a raw I/O failure mid-apply "joins the loom section's grouped transient I/O row".
That row (`contracts/specs/refusal-spec.md`, loom section, `raw I/O`) has the way forward "re-run the refused verb; nothing is mutated", and the transient class's disposition is "halt without mutating state".
A mid-apply failure has already renamed or deleted files, so both halves of that row are false for it.
A re-run cannot help either: D2's own precondition refuses a plan whose Card Index and files disagree (`index-file-mismatch`, `card-numbering`), which is exactly the state a partial apply leaves.
D4's stencil sentence then forbids renaming or deleting a card file by hand, and the Plan-Burler fixer's overlay fix scope runs no git at all (`fixScopeRules` in `internal/burlerengine/prompt.go`), so the agent most likely to hit this has no sanctioned step.
The rubric's attack-surface question requires the new rename and delete edges to state what bounds them; "the plan gate reports the partial state" names a detector, not a bound.

**Fix:**
State in D2 a recovery that leaves the plan consistent, and give it its own row, for example: on a failed step the verb reverses the steps already applied from its in-memory move list, so a transient failure leaves the directory as it was and the grouped row's "nothing is mutated" holds;
a failure of that rollback is a separate correctness-halt row whose way forward names a step the refused agent may take (a lyx verb, or a plain `git checkout HEAD -- <plan dir>` plus the named stray files for a role that may run git).
Add the failure-path case to Acceptance criteria.

### [BLOCKING] The slug bound cites a rule the format does not have

**Location:** `_lyx/discussion/decision-record.md` D2, "`insert-card` refuses a slug the plan format's slug rule rejects."

**Issue:**
`contracts/specs/loom-plan-spec.md` states no slug rule: the Card Index line is `N — <card-slug> — <intent>`, and the parser's `cardIndexLineRe` in `internal/planparser/parse.go` takes the slug as `\S+`.
`cardFileName` builds the file name as `fmt.Sprintf("%02d-%s.md", number, slug)`, so a slug holding `/` lands in a subdirectory and one holding `../..` resolves outside `planparser.PlanDir` once joined.
D2's first bound ("They touch only the current generation's card files … under `planparser.PlanDir`") therefore rests on a check that has no definition, and the new file-write edge has no stated bound.

**Fix:**
Define the slug rule in D2 itself (for example lowercase letters, digits and single hyphens, non-empty), so the bound is checkable; if the format should carry it for every card, say that the plan spec's Card Index section gains it in the same commit.
Add an invalid-slug row with a path separator to the Acceptance criteria refusal list.

### [MEDIUM] The begun-card bound is read without webster's lease

**Location:** `_lyx/discussion/decision-record.md` D2, the begun-card bullet.

**Issue:**
Refusing positions at or before the last begun card is the right bound, and with it a renumber moves only cards `Rebaseline` re-batches as the tail (`rebaselineBatches` keeps exactly the partition up to the last batch with a record).
The bound is a check-then-act, though: webster's begin-batch records a batch under the state-mutation lease (`websterengine.AcquireStateMutation`), and `lyx webster rebaseline` holds that lease across its load, check and save (`internal/webstercli/rebaseline.go`).
D2 names no lease, so a begin-batch between the verb's read and its apply can record a batch holding card `N` and its file hash, after which the verb renames that card: the run record then names a card id the plan no longer has at that number, the desynchronization the bound exists to prevent.
The repo's precedent for plan-directory mutation beside a live run is the "plan-write archive: webster busy" row, which refuses while the run lock is held.

**Fix:**
State in D2 that the verbs hold webster's state-mutation lease from the run-record read through the last apply step, or refuse while a webster run holds its run lock, and add that refusal's row and Acceptance criteria line if the second is chosen.

### [LOW] The --intent value has no bound

**Location:** `_lyx/discussion/decision-record.md` D1 and D2.

**Issue:**
`insert-card` writes `--intent` verbatim into the Card Index line `N — <slug> — <intent>`.
An empty value fails `cardIndexLineRe`'s `(.+)`, and a value holding a newline splits into a second index line, so the verb itself writes an overview the parser rejects, after which D2's precondition refuses every further insert or remove.
D2's refusal list does not cover the flag.

**Fix:**
Add to D2's bounds that `insert-card` refuses an empty or multi-line intent, and add it to the Acceptance criteria refusal list.

### [LOW] The rebaseline open risk is answerable now

**Location:** `_lyx/discussion/decision-record.md` Open risks, first bullet; `_lyx/discussion/support-log.md` Question ledger Q9.

**Issue:**
The code answers it.
Under the loom's Webster row, `autoRebaseline` (`internal/websterengine/autorebaseline.go`) names every changed card file's number itself, so no agent names them.
For a standalone `lyx webster rebaseline`, `Rebaseline` refuses any changed card file whose number is not passed with `--card`, and `changedPlanFiles` counts both the new names and the vanished old names, so every number from `N` to the last card plus one must be named; the moved-card list in the envelope gives them.
The overview change passes because `insert-card` and `remove-card` change only the Card Index, which the overview frame hash excludes (`OverviewWithoutCardIndex`), which is why D2's "never any other overview section" bound matters.
No webster change is needed.
The writer stencil tells the writer to design the full scope now and never defer; this question is a code read, not an operator tradeoff.

**Fix:**
Replace the open risk with a Constraint or Note stating the answer above (the overview-section bound is what keeps rebaseline accepting the edit), and mark Q9 resolved in the ledger.

### [LOW] D2 and D3 name the wrong run-record reading

**Location:** `_lyx/discussion/decision-record.md` D2 ("read the way `loomshed.ValidatePlan` reads it") and D3 ("as `loomshed.ValidatePlan` already does").

**Issue:**
`loomshed.ValidatePlan` reads `websterengine.DoneCards`, which keeps only records terminal with status done.
The begun set is every batch record in `State.Batches`, terminal or not, which is what `DispatchScope` and `rebaselineBatches` use.
Read literally, the two Decisions bound the verbs by the done set, which lets a renumber move an in-flight batch's cards; only the Notes for the plan writer correct this, and the Notes are optional by contract.

**Fix:**
State in D2 that the begun set is the card ids of every batch record webster's run record holds, terminal or not, loaded the way `ValidatePlan` loads the state; keep D3's pointer to the loading only.

### [LOW] The reachability rule over-reaches to every error return

**Location:** `_lyx/discussion/decision-record.md` D5, first paragraph.

**Issue:**
The rule binds "a refusal, error return or guard branch".
An error return includes a wrapped raw I/O failure, which `refusal-spec.md` places in one grouped transient row per section.
Read literally, every card that propagates a write error must name a reaching input and test for it in its `**Intent:**`, which inflates cards with no tie to the gap in issue #516, a refusal unreachable by construction.

**Fix:**
Narrow the rule to refusals and guard branches the card prescribes, or state that a raw I/O error joining its section's grouped row is exempt, and mirror the same scope in the `## Also flag` bullet.

### [LOW] Residual-risk commentary belongs in the support log

**Location:** `_lyx/discussion/decision-record.md` Open risks, second bullet.

**Issue:**
The bullet says a reviewer may still miss a distant upstream rejection.
Under the completeness-before-leanness test it carries no requirement or constraint for the plan writer: the requirement it narrows (a card names its reaching input and the upstream check) is already in D5.
It is deliberation about the design's limits, which `Plan-Write` cannot act on.

**Fix:**
Move the bullet to `support-log.md` (the Interview's fifth turn, or the ledger), leaving D5 unchanged.

## Verdicts on the focus items

- D1/D2 stub claim: holds.
  A heading-only card carries no type label and no `**Intent:**`, which `loom-plan-spec.md` makes `card-type-missing` and `card-missing-field` findings in the format-only set both plan gates run.
- D2 begun bound: right bound, wrong reading in the Decision text (F6) and unguarded against a concurrent begin (F3).
- D2 apply order: the orders hold at every step; the partial-state outcome is unbounded (F1).
- D2 refusal list against Acceptance criteria: each D2 refusal has an Acceptance criteria line (`card-numbering` falls under "index/file mismatch"); the slug and intent edges are unbounded (F2, F4).
- D5 wiring-guard premise: true.
  `refusal-spec.md`'s wiring-guard class covers guards "unreachable from any on-disk state a run can produce", grouped with no per-row test, and `PATTERN-refusal-way-forward` binds only reachable refusals.
- D4 premise: true.
  `loom-rubric-plan-review.md` has a `## Write surface` section, and burler renders the explore step, which carries `{{.rubric}}`, for both the reviewer and the fixer half.
- Open risks: the first bullet is answerable now (F5); the second belongs in the support log (F8).
  Notes for the plan writer carry nothing that needs to move.
