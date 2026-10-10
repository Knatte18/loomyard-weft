---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:106
    summary: The partitioned Rename-old way forward says to Create the new member in each of its files, but a plan can claim one handle once, so a per-file Create is refused handle-collision
---

### [MEDIUM] The partitioned Rename-old way forward names a per-file Create the plan format refuses

**Location:** `_lyx/discussion/decision-record.md:106`, with the acceptance criterion at line 167.

**Issue:** D6 decides that a partitioned Rename old side is refused `rename-old-unresolved`, and that the detail names this way forward: "a Delete of the old member and a Create of the new one in each of its files".
A plan writer or planner reading that literally writes one Create declaration per file.
The plan format allows only one claim per handle.
`checkHandleConsistency` in `internal/planparser/validate.go` reports `handle-collision` when `handleClaims` holds more than one claim for a handle, whether from two Create sub-bullets on one card or across cards.
A handle's unit is its package directory, never a file, because `checkHandleMalformed`'s file-unit rule refuses a `.go` unit.
So "a Create of the new one in each of its files" can't be written as one declaration per file.
The only form the format accepts is one Create declaration of one handle, with each partitioned file as a target of that Create.
After D6, the Create done-check then accepts the partitioned answer.
The text D6 tells lyx to emit is the operator's and the planner's whole guide out of this refusal.
As phrased, it can lead to a second refusal, `handle-collision`.
The same premise D6 gives for keeping the old side Found-only also applies here: partitioned declarations may differ.
A single Create declaration carries one declaration head, and D6 doesn't say whether that holds when the per-file signatures differ.

**Fix:** In D6, state the way forward in a form the plan format accepts.
For example: a Delete of the old member, then one Create of the new member's handle whose targets are each of the old member's files, or an Edit of each file's body.
Make the acceptance criterion at line 167 pin that wording.
If one declaration head can't stand for per-file declarations whose signatures differ, D6 should say so and name only the Edit route for that case.

## Focus directive verdicts

- Rename-old for a partitioned member (focus 1): D6 decides this at lines 105–106.
  The old side keeps `renameDeclSource`'s Found-only rule, which matches `internal/planglyph/handle.go`.
  Scope item 6 (line 19) and the acceptance criterion at line 167 agree with that decision.
  `statusFindings` passes found and multipart today, and `renameDeclSource` then refuses multipart, so "Rename endpoints" at line 104 alongside the Found-only old side mirrors the existing multipart behavior.
  It isn't a contradiction.
  The one defect here is the way-forward wording, which is F1.
- Base context and negated `go1.N`/`gc` (focus 2): this is not a defect.
  Line 97 states the base context: a zero `build.Context` with `Compiler`, `ReleaseTags` and `ToolTags` empty.
  I checked `go/build`'s `matchTag` in the local GOROOT.
  With those fields empty, `name == ctxt.Compiler` never matches a non-empty tag, and the `ToolTags` and `ReleaseTags` lookups always miss.
  So `gc` and `go1.N` count only through `BuildTags`.
  The `boringcrypto` → `goexperiment.boringcrypto` rename and the android/illumos/ios/unix implications follow that source as D6 states them.
  Line 168 covers a `!go1.21` file and a `!gc` file, each beside an unconstrained file, and both stay `glyph-ambiguous`.
  That is correct, because the environment with the tag false selects both files.
- Free-tag cap (focus 3): this is not a defect.
  Line 102 decides the cap as more than four free tags, and line 113 states the enumeration cost: pinned pairs × 2 cgo values × 16 assignments per candidate.
  The Open risks entry at line 152 (five or more free tags stay `ambiguous`) and the acceptance criterion at line 168 (a set naming five free tags is not partitioned) agree with both values.
- Contract-file refusal wording for a recovery writer (focus 4): this is not a defect.
  Today `contractDeleteClause` in `internal/websterengine/contractevidence.go` hard-codes "a fork wrote".
  Line 73 of D3 decides three things.
  The refusal and audit text for an uncleared contract file names the last writer as a fork or a recovery session.
  The refusal-spec rows for a contract file last written by a fork change too: lines 112, 115, 116 and 120 of `contracts/specs/refusal-spec.md`.
  A finding a recovery kept never reads "a fork wrote".
  The acceptance criterion at line 161 pins both wordings, and the Constraints entry at line 118 lists the changed rows under `PATTERN-refusal-way-forward`.
