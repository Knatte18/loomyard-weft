---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:137 (D7, also D1 at :33 and the re-sign bullet at :127)
    summary: D7 strips the explicit-interface qualifier from Owner only for handles; a re-sign arrow names no Owner source and a Rename takes the resolved glyph's owner, which holds the qualifier the head also spells, so quarry Name rejects or misnames every explicit interface member
    fix: In D7, replace the handle-only explicit-interface sentence with one rule for every C# declaration (handle, re-sign arrow, Rename) - Owner is the glyph's owner chain with the trailing components the head spells as the member's interface qualifier removed - and state that a re-sign arrow's Unit and Owner come from D3's parse of the arrow's target. Add an acceptance criterion - under csharp, a re-sign arrow on an explicit interface member such as Ns#Renderer.IProvider<A>.Value mints its own target id.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:142 (D7)
    summary: A C# Rename of one name in a multi-declarator field, const or event field derives a head that declares several symbols, which quarry Name refuses, and D7 drops the one-name synthesis that exists for exactly this case without stating the bound
    fix: In D7, replace the sentence that gives a C# const or field the plain substitution with - a C# field, const or event field derives a one-declarator head - the resolved signature through the declared type, followed by the new identifier, with = default appended for a const - since quarry gives every declarator of one declaration the whole declaration as its signature. Add to the Rename acceptance criterion a const declared as public const int A = 1, B = 2 whose A is renamed.
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:181 (Open risks)
    summary: The open risk on a not-yet-existing slash-free root path self glyph hands downstream a fact that quarry's docs and today's Create check already settle
    fix: Delete the Open risks bullet that starts Under csharp, a slash-free root path self glyph. Add to D3 - such a target answers not_found with unit not_found, per quarry v0.4.0 docs/glyph.md section 5 (if none matches, not_found, with unit found when any accepting alphabet found the unit), and the Create check passes a not_found answer with at most the informational create-new-unit, exactly as for a missing Go file today.
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:92 (D4 bound)
    summary: D4's bound says a go plan whose only C# refs are namespace self glyphs is let through, but under go such a glyph is a Go path self glyph and path-missing refuses it when no directory of that name exists
    fix: Replace the D4 bound sentence starting A go plan whose only C# refs are namespace self glyphs with - a go plan whose only C# refs are namespace self glyphs, such as Norce.Models#, is not caught by this finding - path-missing refuses it when no directory of that name exists, and resolve answers glyph-ambiguous when one does.
  - id: F5
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:134 (D7 record-batch bullet)
    summary: D7 does not place its record-batch rewrite relative to DetectDrift or say how ScopeGuard sees the minted id, so every C# re-sign that changes a parameter yields informational rename-candidate notices on later cards and a scope-outside-plan notice for its own minted id
    fix: In D7's record-batch bullet, state that the old-to-minted rewrite runs before DetectDrift reads the pending plan, so no later card still names the old glyph, and that the re-sign card's minted id joins ScopeGuard's target union for that card.
  - id: F6
    severity: LOW
    class: scope
    location: _lyx/discussion/decision-record.md:129 (D7 resign-overload-collision)
    summary: resign-overload-collision checks the minted id only against members that already resolve found, not against a Create handle or Rename new handle elsewhere in the plan that binds to the same id, so that collision surfaces mid-run
    fix: In D7, extend resign-overload-collision to refuse a minted id that equals the canonical id any card's Create handle or Rename new handle binds to, and name in its detail the card that declares it; extend the overload-collision acceptance criterion with that case.
---

### [MEDIUM] Owner for C# re-sign arrows and Renames keeps the explicit-interface qualifier

**Location:** `_lyx/discussion/decision-record.md:137`, D7; D1 at `:33`; the re-sign bullet at `:127`.

**Issue:** D1 says a C# declaration sets `Owner` "(D7)", and D7 defines it only for a handle: the owner chain without the interface qualifier.
The re-sign bullet names no `Owner` source, and the Rename bullet takes "the owner from the resolved glyph".
Quarry v0.4.0 spells an explicit interface implementation with the qualifier inside the owner chain (`docs/glyph.md` §3: `void System.IDisposable.Dispose()` is `Renderer.System.IDisposable.Dispose()`), and the head spells it again.
Quarry's `Name` (`internal/engine/name.go`, `nameCSharpContainer`) refuses an owner component with type arguments, so `Renderer.IProvider<A>` as `Owner` is `invalid_container`.
For `Renderer.System.IDisposable` it builds class shells for all three components and the head adds the qualifier again, so the minted id differs in owner from the target, and D7's apart-from-parameter-list rule reads that as `resign-head-mismatch`.
So a re-sign on any explicit interface member fails, and its detail ("keep the member's parameter list") points the wrong way.
Round 1's F9 raised the same qualifier problem, and its fix reached only the handle bullet.

### [MEDIUM] Multi-declarator C# field, const or event-field Rename cannot be named

**Location:** `_lyx/discussion/decision-record.md:142`, D7.

**Issue:** Quarry mints one symbol per declarator and gives each one the whole declaration as its node (`internal/engine/csharp.go`, the `field_declaration` and `event_field_declaration` arms), so `Signature` is `public int a, b` for both `a` and `b`.
D7's substitution then yields `public int c, b`, which declares two symbols.
Quarry's `Name` refuses that with `several_declarations` (`internal/engine/name.go`, `outermostSymbol`).
The Go var/const synthesis in `renameDeclSource` exists for exactly this case ("can declare several names and which quarry.Name rejects then").
D7 switches it off for C# and says "a C# const or field takes the same substitution as any other C# member", with no bound naming what then fails.

### [MEDIUM] Open risk defers a settled fact about a missing root self glyph

**Location:** `_lyx/discussion/decision-record.md:181`, Open risks.

**Issue:** "A Create card's expectations must hold for that answer" leaves Plan or an implementer to find out a fact the design rests on, which the rubric flags as a deferred research question.
The fact is already settled.
Quarry v0.4.0 `docs/glyph.md` §5 says that when no alphabet matches, the answer is `not_found`, with `unit: found` only when some accepting alphabet found the unit.
Neither the missing file nor the undeclared namespace is found, so the answer is `not_found` with `unit: not_found`.
`createFindings` in `internal/planglyph/create.go` passes that with only the informational `create-new-unit`, the same as for a missing Go file today.

### [LOW] D4's bound misstates what happens to namespace self glyphs under go

**Location:** `_lyx/discussion/decision-record.md:92`, D4 bound.

**Issue:** Under `go`, `Norce.Models#` parses as a Go path self glyph.
`checkPathMissing` (`internal/planparser/validate.go`) maps it through `diskPathForRef` and reports `path-missing` on an Edit, Delete, Move or Prosa ref or a Uses entry when no `Norce.Models` directory exists.
When such a directory does exist, quarry answers `ambiguous` (`docs/glyph.md` §5).
So the plan is refused either way, not "let through".
The bound misdescribes what the plan writer will see.

### [LOW] Re-sign record-batch ordering against DetectDrift and ScopeGuard unstated

**Location:** `_lyx/discussion/decision-record.md:134`, D7.

**Issue:** For C#, quarry's delta never treats a parameter change as an exact-tier rename: it is a "same name with different parameters" evidence-tier candidate (`docs/glyph.md` §5, the `delta` paragraph).
`DetectDrift` then gives an informational `rename-candidate` on every pending card that still names the old glyph, and that detail tells the reviewer the rename-versus-delete call is theirs.
D7 does not say whether its record-batch rewrite runs before or after `DetectDrift`.
`ScopeGuard`'s union holds only the card's old glyph (`targetGlyphUnion`), so the delta's created minted id draws `scope-outside-plan` unless the card also lists the file.
Both findings are informational, but every such C# re-sign would raise them for an outcome the plan asked for.

### [LOW] Overload collision misses a plan-declared handle

**Location:** `_lyx/discussion/decision-record.md:129`, D7.

**Issue:** The collision check reads only "already resolves found".
A plan that re-signs `Draw(int)` into `Draw(double)` and also creates `Draw(double)` through a handle passes the plan gate.
Both the minted id and the handle's canonical id answer `not_found` at plan time, and `handle-collision` counts only handles.
The duplicate surfaces only at the later card's dispatch, as `create-already-exists`, or as a build failure.

## Focus verdicts

- Interface-method test and receiver synthesis: D7 has `isInterfaceMethod` answer false for every C# glyph symbol.
  That turns off `resign-interface-method` in `plangate.go` and the `func (Owner)` synthesis in `renameDeclSource`, which sits inside `if isInterfaceMethod(sym)`.
  `renameSignature`'s `func (` prefix test never matches a C# head.
  Not a defect; the owner gap is F1.
- Re-sign resolution after an earlier re-sign: D7 runs `Name` on re-sign declarations before the batched resolve and starts a new batch at the first later card that names the target.
  Record-batch resolves the minted id (`resign-not-done`) and rewrites old to minted plan-wide through `planparser.RewriteRefs`, so the mechanism does not rely on `DoneChecks` or `DetectDrift`.
  The bounds on parameter-list presence and overload collision are both stated.
  The residue is F5 and F6.
- Rename identifier, arity, const head and substitution: D7 takes the identifier from D3's parse with the arity stripped, never from `HandleIdentifier`.
  It scopes the var/const synthesis to Go and states the declared-name token rule.
  The multi-declarator gap is F2.
- D3/D4 gates: D4's resolve half catches a `go` plan whose glyphs parse under both alphabets and resolve into `.cs` files.
  D3's `rename-namespace-pair` refuses a pair of two C# namespace self glyphs.
  D3's bound lists `redundant-file-target` and `planparser.RefCovers`, and `ScopeGuard` receives the plan.
  The D4 bound wording is F4.
