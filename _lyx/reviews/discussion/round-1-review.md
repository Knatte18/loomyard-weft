---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:52
    summary: D3 and D5 say the new checks never run at dispatch, but the pure tier and the delete-before-reference seam they extend are shared with ValidateDispatch, so the bound is false and the dispatch acceptance criterion cannot hold as written
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:34
    summary: D1 drives the D3 redundancy check and the D5 caller coverage from the first card, but D7 builds them later, and the record never says when those rows start passing or that the first card must stay green
  - id: F3
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:36
    summary: D1 and D7 require cards for fixes to defects the test exposes, but the plan is written before the test exists, and the record names no mechanism that turns an exposed defect into a card
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:70
    summary: D5 says caller coverage reuses the deleteorder scan over each file in the tree, but that scan only reads later cards' Edit regions, and the file set and the under-reporting cases are left unbounded
  - id: F5
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:68
    summary: D5 re-sign arrow states no bound on a head that names the right member but misstates the signature, and names no consumer or check of the head
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:71
    summary: The no-double-report rule and its acceptance criterion cite delete-before-reference, which exists for Delete only, so a later-card reference to a Rename or re-signed member has no stated outcome
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:7
    summary: The Goal promises the same verdict at Plan-Write and at dispatch, while D3 and D5 deliberately give the plan gate verdicts that dispatch never gives
  - id: F8
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md:44
    summary: D2 says webster runs cards strictly one at a time, but webster runs batches one at a time, one fork per batch, and recover-batch re-runs a whole batch
  - id: F9
    severity: NIT
    class: scope
    location: _lyx/discussion/decision-record.md:102
    summary: Auto-mode assumptions, Open risks and Notes restate decisions or process facts the plan writer cannot act on, and belong in the support log or nowhere
---

### [BLOCKING] D3 and D5 plan-gates-only bound is not implementable on the shared seams

**Location:** `_lyx/discussion/decision-record.md:52-54` (D3), `:70-74` (D5), `:121` (acceptance).

**Issue:**
D3 puts the package-level redundancy pairings in `planparser` and says they run in `ValidateFormat`, `Validate`, `ValidateRework` "and never in `ValidateDispatch`".
`planglyph.ValidateDispatch` itself calls `planparser.ValidateFormat` (`internal/planglyph/planglyph.go`, the loop over `convertAll(planparser.ValidateFormat(...))`) and keeps every finding whose card is not completed.
`planparser.validate` is also where `syntacticContainment`, the check D3 extends, is dispatched (`internal/planparser/validate.go`).
A new `planparser` check therefore runs at dispatch for every pending card, and a blocking one makes `begin-batch` return `ErrPlanDrifted`.
So D3's bound "dispatch never refuses on redundancy" is false for the pure pairings, and the acceptance criterion "the same plan gets no redundancy finding from `ValidateDispatch`" fails on them.

D5 has the same defect.
It puts caller coverage "beside `delete-before-reference`", which `LaterDeleteReferences` emits from `resolvePass`, and `resolvePass` is shared by `ValidateFormat`, `Validate` and `ValidateDispatch`.
D5 also says the check "never runs at dispatch", and no D5 acceptance criterion pins that.

The pure pairings are state-stable: D3 counts a `plan:` handle as a member of its declared unit, so canonicalization at `begin-batch` does not change the verdict.
Only the file-versus-member pairing in `planglyph`, and caller coverage, depend on state that moves.
The record leaves open how a check is kept out of `ValidateDispatch`, and which of the pure pairings may run there.

**Fix:**
Decide the seam in the record.
Either name a plan-gate-only mode that `ValidateDispatch` does not set for both `planparser` and `resolvePass`, or state that the pure pairings may run at dispatch because they are state-stable and keep only the resolve-backed pairing and caller coverage plan-gate-only.
Correct the D3 bound and the acceptance criterion to match, and add a D5 acceptance criterion that `ValidateDispatch` reports no caller-coverage finding after the reference set shrinks.

### [MEDIUM] D1's first-card test depends on checks D7 builds later

**Location:** `_lyx/discussion/decision-record.md:34`, `:86`.

**Issue:**
D1 lists the D3 redundancy check and the D5 caller coverage as steps each test row is driven through, and puts the test in the plan's first cards.
D7 orders D2, D4, D3 and D5 after the test and its fixes.
The order is satisfiable only if the rows exercising a check land with that check, but the record does not say so.
Read literally, the first card commits rows that fail because a check does not exist yet, or rows that omit the checks and are extended later.
Webster gates each card on `go build ./... && go test ./...` plus the integration step, so a first card with red rows cannot land.
Rows for the retired containment checks have the mirror problem: the `rework-generations` shape row fails on `containment-file-overlap` until D2 lands.

**Fix:**
State in D1 or D7 that the first card's rows assert the chain as it stands, that the rows for D2's retired containment, D3 and D5 are added or flipped by their own cards, and that no card commits a failing row except a quarry-rooted row asserting quarry's current answer.

### [MEDIUM] No mechanism turns an exposed defect into a card

**Location:** `_lyx/discussion/decision-record.md:36`, `:86`, `:112`.

**Issue:**
D1 says every row that fails because of loomyard code "is fixed in a card of this plan", and D7 lists "fixes D1 exposes" as a block of cards.
The plan is written and gated before any card runs, so the Plan producer cannot know which rows fail.
A webster run does not add cards, and `planparser`'s amendment log records glyph substitutions only.
The record leaves open whether the plan writer predicts the defects during exploration, writes one catch-all fix card inside the test card, or relies on a rework.
Open risks admits the plan "can become large" but names no way for it to hold a defect nobody has found yet.

**Fix:**
Decide the mechanism in D1 and state it in the record.
For example, the test card carries the fixes its own rows expose, inside its card's scope, or the plan writer runs the chain on the sample shapes during exploration and writes one fix card per defect found.

### [MEDIUM] D5's reference scan is not the existing scan and its limits are not stated

**Location:** `_lyx/discussion/decision-record.md:70-73`, `:111`.

**Issue:**
D5 requires that "each file in the tree that references" a deleted, renamed or re-signed member be covered, and says detection "reuses the existing `deleteorder.go` identifier scan".
`LaterDeleteReferences` only searches the Edit regions of later cards' targets, and "code outside a later card's Edit targets is never searched".
Only its per-symbol regexes are reusable; a tree-wide file walk is new, and the record says nothing about which files it reads.
Open: whether `_test.go` files, `testdata`, generated files, files behind other build tags and non-Go files are searched, and whether a Create target from an earlier card counts.

The attack-surface question for a new blocking gate is also half answered.
Open risks covers over-reporting only.
The scan matches the package-qualified form by the package clause name of the declaring file, so a caller importing under an alias or a dot import is not matched, and the finding lets that caller through.
Nothing states that gap or what bounds it.

**Fix:**
State the file set the scan reads, and add the under-reporting cases (aliased and dot imports at least) and their bound to D5 or Open risks, with the way a planner covers an alias-imported caller.

### [MEDIUM] The re-sign arrow's head has no stated bound or consumer

**Location:** `_lyx/discussion/decision-record.md:68-69`, `:123`.

**Issue:**
D5 adds a grammar edge: an `Edit` bullet `` `<glyph>` -> `<head>` ``.
The head must parse and "name the same member as the glyph".
A head that names the right member but misstates the signature passes, and the record does not say what that lets through, which downstream step reads the head, or whether record-batch or a done-check compares it with the built declaration.
If nothing reads the head, the record should say the arrow is a marker whose head is only parse-checked, so a wrong head is harmless by construction.
Also unstated: the outcome of the arrow on a non-member `Edit` target (a file path) or on a method whose receiver the head changes.

**Fix:**
State in D5 what the head is used for, what a misstated head can let through, and what bounds it.
State what the arrow does on a target that is not a member glyph.

### [LOW] The no-double-report rule is Delete-only

**Location:** `_lyx/discussion/decision-record.md:71`, `:125`.

**Issue:**
D5 says a reference in a later card's Edit region "keeps the existing `delete-before-reference` finding and is not reported twice", and the matching acceptance criterion repeats it.
`delete-before-reference` exists for `Delete` targets only.
For a `Rename` Old side or a re-signed `Edit` member, a caller in a later card's region matches neither "covered by the same or an earlier card" nor an existing finding, so a plan writer cannot tell whether caller coverage reports it.
By D5's own rule it does, as a blocking finding.

**Fix:**
Scope the sentence and the criterion to `Delete`, and state that a later-card reference to a renamed or re-signed member is a caller-coverage finding.

### [LOW] The Goal overclaims verdict identity

**Location:** `_lyx/discussion/decision-record.md:7`, `:54`, `:74`.

**Issue:**
The Goal says plan validation "gives the same verdict at Plan-Write and at dispatch".
D3 and D5 create blocking checks that run at the plan gate and never at dispatch, so the verdicts differ by design.
The intended property is that no check changes verdict mid-run, which is what D2 to D5 enforce.
A plan writer reading the Goal literally would try to run the new checks at dispatch.

**Fix:**
Reword the Goal to the property the decisions deliver, for example that no check changes its verdict between the plan gate and dispatch because of state the run itself changes.

### [NIT] D2's bound names the wrong unit of sequencing

**Location:** `_lyx/discussion/decision-record.md:44`.

**Issue:**
`internal/websterengine/doc.go` states the loop forks one implementer per execution batch, one fork at a time.
Cards inside a batch go to the one fork, and `recover-batch` re-runs the same batch with a cold strand, possibly beside a still-running fork's late report.
None of these paths pairs two different cards in parallel, so D2's bound holds for dispatch, rework (a plan-time gate), rebaseline (no dispatch) and recover-batch.
The sentence "webster runs cards strictly one at a time" is the imprecise part.

**Fix:**
Say that webster runs batches one at a time, each batch to one fork, and that a retried batch re-runs the same cards.

### [NIT] Restatements and process facts outside the support log

**Location:** `_lyx/discussion/decision-record.md:104-106`, `:111`, `:113`, `:135`.

**Issue:**
No requirement is carried only by these lines, which the completeness test needs before relocating.
The first Auto-mode bullet is a process fact already in the support log's Interview and Question ledger, and the third is a conclusion, not an assumption the plan writer relies on.
The first and third Open risks restate D5's bound and D1's quarry-row rule.
The fourth Notes bullet repeats D2's bound with a pointer.

**Fix:**
Drop the first and third Auto-mode bullets and the Notes bullet on webster, and cut the restated halves of the first and third Open risks.

## Focus verdicts

1. D1 against D7: not satisfiable as written, see the second finding. The first card must stay green, so D2, D3 and D5 rows land with their checks.
2. D4 against interview item 4: not a defect. `checkHandleConsistency` excludes the declaring card's own Create bullet from the references that count, so the check does fire today for a handle only its own card uses, which D4's rationale says. Interview item 4 and the rejected alternative describe the board's proposed exemption, which would make the check dead code. The decision record's account is true on its own.
3. D2's bound: holds on every dispatch path, with the wording imprecision in the NIT above.
4. D5's new edge and gate: bounds missing, see the re-sign finding and the reference-scan finding.
5. Plan-gates-only: the record says no finding is surfaced after the plan passes the gate, which is the intended design, but the seams cannot deliver it and the D5 acceptance criterion is missing; see the BLOCKING finding.
6. Completeness-before-leanness: no gap. The card-order basis (D5), the handle-as-member rule (D3) and the informational severity for methods and fields (D5) are all in the decision record.
7. Relocation: only the NIT above.
