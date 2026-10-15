---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: _lyx/discussion/decision-record.md:71
    summary: The PATTERN-ref-shape-registry constraint needs a planparser export that Scope In does not name, because no exported planparser function tells a member glyph from a package self glyph
    fix: In decision-record.md, add a Scope In bullet for internal/planparser that names the new exported accessor ScopeGuard calls to tell a member glyph or member plan handle from a package self glyph or directory path, for example an exported form of refMember in targetdirs.go. Extend the Constraints bullet at line 71 to say that RefScope cannot make this distinction, since it returns isFile false for both, and that RefCovers does not apply because its directory cover reaches subdirectories.
  - id: F2
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:31
    summary: D1 does not say which file decides file and package coverage for a symbol whose file differs between the delta's sides, so a symbol moved out of a targeted file into an untargeted one could leave no finding at all
    fix: In D1, after the package-target bullet, add a sentence fixing the file each touched symbol is judged by. Recommended - a Modified symbol is covered only when every Before and After file is covered, a Renamed pair falls back to file and package coverage only when both From.File and To.File are covered, and Created and Deleted use their own File. Add a matching row to Acceptance criteria - a symbol moved from a file target into an untargeted file is still reported.
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:47
    summary: D2's Uses bullet says any change to a member a card only reads is a deviation, but under D1 a Uses member inside a file or package a batch card targets is in scope, so the fork prompt would state a rule the guard does not apply
    fix: Replace decision-record.md line 47 with - `**Uses:**` entries add no coverage, so a change to a member a card only reads is a deviation unless a target of some card in the batch covers it.
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:87
    summary: Acceptance criteria and Decisions do not match one-to-one - the second prompt criterion bans naming a glyph or changes-files union, which no Decision states, and D2's Uses rule and D1's two package-target shapes have no criterion
    fix: In decision-record.md, either add that ban to D2's Decision text or drop the "names a glyph or changes-files union" clause from the prompt criterion at line 91. Reword the criterion at line 87 to name both package target shapes, a package self glyph and a directory path. Add a criterion that the step-2 rule states that Uses entries add no coverage, worded to match the fix for F3.
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:63
    summary: D3 keeps two facts and replaces the comparison but does not say what happens to the paragraph's third sentence, the flat-target-set and multi-label sentence that begins "This union is defined", which has no referent once the union description is gone
    fix: In D3, add a sentence saying the plan spec's sentence beginning "This union is defined over each card's flat target set" is dropped, since the ScopeGuard pointer covers it, or is reworded to say that ScopeGuard reads each card's flat target set.
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:39
    summary: D1's rationale sentence "That is the bug the board entry names, now produced by the guard rather than the fork" is history that belongs in the support log, and it overclaims because member-glyph targets will still list their own files when an untargeted sibling in them changes
    fix: Delete decision-record.md line 39. It carries no requirement, since lines 38 and 40 keep the reason and the bound. Turn 1 and the Turn 3 premises entry in support-log.md already record the history.
---

### [MEDIUM] Ref-shape constraint forces a planparser change Scope In omits

**Location:** `_lyx/discussion/decision-record.md:71`

**Issue:** D1 makes coverage depend on target shape: a member glyph is exact, while a file, package self glyph or directory path covers its members.
The constraint routes that distinction through `internal/planparser`.
But `planparser.RefScope` (`internal/planparser/targetdirs.go:93`) returns `isFile=false` with the package directory for both a member glyph and a package self glyph, so it cannot draw the line.
The only classifier that can is `refMember` (`targetdirs.go:171`), and it is unexported.
`RefCovers` is exported but does not match D1: its directory cover covers subdirectories, and its file cover covers any glyph in the file's package.
No other exported `planparser` function classifies member against self.
So D1 cannot be built without a new `planparser` export, and Scope In lists only `ScopeGuard`, the stencil and the plan spec.
The Notes bullet says the symbol check "must not" count a member glyph's package directory but does not say how to tell the two apart.

### [MEDIUM] D1 leaves the deciding file open for a symbol that changes file

**Location:** `_lyx/discussion/decision-record.md:31`

**Issue:** D1 widens the guard to "the file the symbol lives in", but a Modified symbol carries `Before` and `After` sides whose files can differ, and a Renamed pair carries `From.File` and `To.File`.
Today `ScopeGuard` (`internal/planglyph/scope.go`) reports a Modified symbol under `After[0].File` (`Before[0].File` when `After` is empty) and a rename under `To.File`.
D1 does not say whether coverage uses one side, either side or both.
With either-side coverage, a card with a file target could move a member into an untargeted file and leave no finding: the symbol check passes on the old file, and `filesWithSymbols` keeps the new file out of the symbol-less check.
Judging only the after side would let the reverse move hide the source file's change.
This is the bound the attack-surface rule asks D1 to state, and it is where the rename shape in the round's focus can slip past it.

### [MEDIUM] D2's Uses bullet contradicts D1's file and package coverage

**Location:** `_lyx/discussion/decision-record.md:47`

**Issue:** D2 tells the fork that "a change to a member a card only reads is a deviation" without condition.
Under D1, a member named only in `**Uses:**` that sits in a file target of any batch card, or directly in a package target's directory, is in scope, and the guard reports nothing.
D2's own first bullet says a deviation is a change outside the targets "as `ScopeGuard` counts them", so the second bullet disagrees with the first.
The prompt would then call a covered change a deviation.

### [LOW] Acceptance criteria and Decisions are not one-to-one

**Location:** `_lyx/discussion/decision-record.md:87`

**Issue:** The prompt criterion at line 91 requires that neither prompt "names a glyph or changes-files union".
No Decision states that ban: D2 states the rule's content, and D3 moves the union wording out of the plan spec only.
In the other direction, no criterion covers D2's `**Uses:**` rule.
D1 names two package-target shapes, a package self glyph and a directory path, but the criterion at line 87 says only "package target directory", so a test can pass while covering one shape.
Every other D1, D2 and D3 behavior has a matching criterion: file, package and member coverage, the unchanged handle, rename and symbol-less verdicts, the three sanctioned deviations, the `FAILED` statement and the plan spec pointer.

### [LOW] D3 is silent on the paragraph's third sentence

**Location:** `_lyx/discussion/decision-record.md:63`

**Issue:** The plan spec paragraph under `## Deferred / forward-compat` (`contracts/specs/loom-plan-spec.md`) has three parts: the union description, the websterengine pointer, and "This union is defined over each card's flat target set …, so it is unchanged by multi-label".
D3 names what it keeps (two facts) and what it replaces (the comparison) but not the third sentence.
Once the union description is gone, "This union" has no referent.
D3's premises hold in the code: the paragraph describes a `UnitPath`-mapped changed-files comparison that `ScopeGuard` does not make, and the websterengine pointer is on the next line.

### [LOW] D1 rationale carries history and overclaims

**Location:** `_lyx/discussion/decision-record.md:39`

**Issue:** "That is the bug the board entry names, now produced by the guard rather than the fork" is history, and the support log's Turn 1 and Turn 3 already record it.
It is also wider than D1 delivers.
The board entry's symptom is files that house the batch's own target glyphs.
Under D1, a card with a member-glyph target still gets its own file listed when it changes an untargeted sibling, which D1 intends.
The completeness test passes: lines 38 and 40 keep the reason and the bound, so deleting the sentence loses no requirement.

## Verified, no finding

- D1's premise holds: `targetGlyphUnion` builds an exact-match set of `resolveKeyFor`-normalized targets, so a member of a file or package target never matches and is reported today.
- The Notes claim holds: `targetsCover` takes `RefScope(nil, t)`, and for a Go member glyph that returns the package directory with `isFile=false`, so the symbol-less file check covers a member glyph's whole package directory.
- D1's package bound, files directly in the target's directory, matches `targetsCover`'s `scope == pathpkg.Dir(path)`.
  Go is the only plan language (`planLanguage`), and `Glyph.UnitPath` for Go is the package unit, so no member glyph resolves to a file scope.
- A `plan:` handle's body is a member glyph (`OwnedHandles`, `HandleMember`), so the handle row cannot widen into package coverage.
  `Uses:` never reaches `card.Targets` (`parse.go`), so the guard side stays bounded.
- D2's member, file, package and rename wording matches D1.
  All three sanctioned deviations exist with the stated bounds: step 3's "Change only a file the guard step's failure names", `webster-fragment-guard-fix.md`'s "fix that file under that card's commit", and the self-fix section's two-pronged test and `{{.self_fix_cap}}` bound.
- The record holds no other deliberation narrative that belongs in the support log.
