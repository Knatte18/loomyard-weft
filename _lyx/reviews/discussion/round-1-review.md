---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md D5 and D7 (re-sign, Rename)
    summary: isInterfaceMethod is a Go-signature test that stays on under csharp and classes every C# method as an interface method, so every C# method re-sign is refused and every C# method Rename head is corrupted
    fix: In D5, add resign-interface-method (internal/planglyph/plangate.go isInterfaceMethod) and the interface-method receiver synthesis in renameDeclSource (internal/planglyph/handle.go) to the Go-only checks, or state in D7 that isInterfaceMethod reads the symbol's language and answers false for every C# symbol; add an acceptance criterion that a C# method re-sign raises no resign-interface-method and a C# method Rename derives a head with no func receiver.
    origin: lens:existing-paths
  - id: F2
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md D7 (re-sign bullet)
    summary: D7 says the done check and drift expect a C# re-sign's minted id, but no existing path carries it, so a later card naming the old glyph meets glyph-not-found at its own dispatch
    fix: In D7, name the mechanism, for example after CanonicalizeHandles has the C# re-sign's minted id, rewrite the old glyph to the minted id plan-wide through planparser.RewriteRefs (the same rewrite handles use), after the re-sign card lands; or, if no rewrite is wanted, state the bound (a later card may not name a C# member an earlier card re-signs) and add a plan check that refuses it. Drop the claim that the done check and drift expect the minted id unless a Decision adds that rule, and fix the matching acceptance criterion to the chosen mechanism.
    origin: lens:bound-breaker
  - id: F3
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md D7 (Rename bullet)
    summary: C# Rename derivation still runs Go string work (last-dot identifier split, new-side arity suffix kept, const head synthesis, first-word substitution), so C# Renames of methods, generic types, consts and same-named members fail or rename the wrong token
    fix: "In D7's Rename bullet, state that under csharp the new identifier comes from D3's helper parsing the new handle's body, with the arity suffix stripped (Crate<> gives Crate), never from planparser.HandleIdentifier's last-dot split; that the Go var/const head synthesis in renameDeclSource applies only to Go symbols; and that the substitution replaces the declared name token, not the first whole-word match in a signature that carries attributes and a type (public Color Color { get; }). Add acceptance rows for a C# const Rename and a property whose type shares its name."
    origin: lens:bound-breaker
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D4 (Decision, Bound, Rationale)
    summary: A C# plan left on the default go whose glyphs the Go alphabet also parses (types, properties, fields, namespace self glyphs) raises neither plan-language-mismatch nor glyph-malformed, resolves found through quarry's C# alphabet, and fails later with no language hint
    fix: In D4, either widen plan-language-mismatch to fire under go when a resolve answer for a plan target names a .cs file (or carries a C#-only Kind) and no target names a .go file, or state this case in D4's Bound as let through. Then correct the Rationale's claim that D4 covers a planner whose cards name only member glyphs.
    origin: lens:bound-breaker
  - id: F5
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D3 (rename-pair sentence) and Scope Out (namespace Rename)
    summary: Under csharp a pair of two namespace self glyphs is classed as a file pair by D3's rule, so a namespace Rename that Scope puts out passes the gate and runs git mv
    fix: In D3, restrict the file-pair rule under csharp to pairs whose two sides each have a Go-alphabet reading with a file extension or a slash (path self glyphs), and add a refusal for a pair of two C# namespace self glyphs that names the way forward (rename the namespace through Edit cards); add an acceptance row for it.
    origin: lens:interactions
  - id: F6
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D3 (Bound, and every-site sentence)
    summary: D3's bound claims to list all it lets through but omits redundant-file-target and the batcher's rename/delete boundaries (RefCovers), and ScopeGuard calls RefMember and RefScope with a nil plan, so it reads the default Go alphabet whatever the plan says
    fix: "In D3's Bound, add: redundant-file-target is not raised for a slash-free self glyph read as C#, and planparser.RefCovers maps no C# member or namespace glyph to a scope, so a later Delete or Rename of a file holding an earlier card's C# member target draws no batch boundary (or decide that RefCovers covers a C# member by its resolved file and say so). Add to D3 that ScopeGuard receives the plan, or its language, so its RefMember and RefScope calls go through the helper."
    origin: lens:existing-paths
  - id: F7
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md D3, D4 and Constraints
    summary: D3 says no caller parses with a language of its own and the Constraints say a C# glyph is never read as a path, while D4 reads a ref D3 reads as C# (Program.cs#) in its Go-alphabet path reading
    fix: In D3, state that planparser also exports the Go-alphabet path reading of a canonicalized path entry, and that D4 is its only caller; reword the PATTERN-glyph-conversion-chokepoint constraint line to allow that one reading inside planparser.
    origin: lens:interactions
  - id: F8
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md D7 (re-sign bullet)
    summary: The relaxed re-sign guard states no bound, so it accepts a re-sign onto an existing overload's glyph and a head whose minted id drops or adds the parameter list (a method re-signed into a property)
    fix: In D7's re-sign bullet, add a Bound stating that the minted id must keep the target's parameter-list presence (both with or both without parentheses), and that the plan gate refuses a C# re-sign whose minted id already resolves found in the batched resolve, naming the overload it collides with.
    origin: lens:bound-breaker
  - id: F9
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md D7 (Create-handle bullet, Owner)
    summary: For an explicit interface member, D3's parsed owner chain carries the interface qualifier, and passing it whole as Owner makes quarry Name fail invalid_container
    fix: In D7, state that Owner is the owner chain without the explicit-interface qualifier that the declaration head spells itself, or state that a C# Create handle for an explicit interface member is out of scope and refused.
    origin: lens:existing-paths
  - id: F10
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D6 (third bullet) and Acceptance criteria (README.md row)
    summary: No LoomYard code reads a self glyph's listing, so D6's listing rule governs an empty set and the criterion's clause its listing is read has nothing to test
    fix: Delete D6's third bullet and the words and its listing is read from the README.md acceptance criterion; if a consumer is wanted, name it in a Decision instead.
    origin: handler
  - id: F11
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D1
    summary: D1 takes a re-sign arrow's Lang from the resolved glyph, but the re-sign declaration is built from D3's parse of the target with no resolve answer, and D7 takes its owner from that parse
    fix: In D1, change the re-sign arrow's Lang source to the language of D3's parse of the arrow's target.
    origin: lens:interactions
  - id: F12
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D5 and D6 (build-constraint partitioning)
    summary: D5 gates partitioning on the plan's language while D6 gates it on the candidate's language, and the two disagree for mixed candidates, where a .cs file name suffix such as _windows can be read as a GOOS constraint
    fix: State one rule in D6 and point D5 at it, for example that classifyAmbiguity partitions only .go candidates under any plan language, and that a candidate set holding a .cs file is never partitioned.
    origin: lens:interactions
  - id: F13
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D5 (Rationale and Bound)
    summary: The verify-package checks key on Verify command strings including the language-neutral lyx gate test, not on .go files, so the rationale is false for them and the bound omits that a C# card's lyx gate test package argument goes unchecked
    fix: In D5, either keep verify-package-missing on under csharp for lyx gate test arguments, or add to D5's Bound that a C# card's Verify package arguments are not checked and correct the rationale to say so.
    origin: lens:bound-breaker
  - id: F14
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md Notes for the plan writer (second bullet)
    summary: The PATTERN background file update is a requirement, but it sits in the optional Notes section, which is never a completeness requirement
    fix: Move the second Notes bullet into Scope In (the pattern background file pattern/PATTERN-glyph-conversion-chokepoint.md gains the alphabet rule and the Go-unit-alphabet path rule) and delete it from Notes.
    origin: handler
  - id: F15
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md Scope (per file, by extension paragraph) and Open risks (last bullet)
    summary: The Scope paragraph restates D2, D3 and D4, and the last Open risk restates D3's bound, which is deliberation and repetition rather than a requirement
    fix: Cut the three-sentence Scope paragraph beginning The board entry asks for the alphabet, after checking the departure from per file, by extension is recorded in support-log.md Turn 2 (it is); cut the Open risk C# batch estimates undercount, since D3's Bound carries it.
    origin: handler
---

### [BLOCKING] isInterfaceMethod misfires on every C# method

**Location:** `_lyx/discussion/decision-record.md` D5 and D7; evidence `internal/planglyph/plangate.go:91-123`, `internal/planglyph/handle.go:145-148`, quarry `internal/engine/csharp.go:312`.

**Issue:** `isInterfaceMethod` reports any `KindMethod` whose signature does not start with `func (`.
A C# Signature is verbatim source from the declaration's first byte, so it never starts that way.
D5 leaves `planGatePass`'s `resign-interface-method` on, so every C# method re-sign is refused.
`renameDeclSource` prefixes `func (<Owner>) ` to every C# method signature before `quarry.Name`, so every C# method Rename fails `handle-name-failed`.
The D7 re-sign and Rename acceptance criteria cannot pass.

### [BLOCKING] A C# re-sign's minted id reaches no done check, drift or later card

**Location:** `_lyx/discussion/decision-record.md` D7 re-sign bullet; evidence `internal/planglyph/handle.go:228-283`, `internal/planglyph/donecheck.go` `DoneChecks`, `internal/planglyph/drift.go` `DetectDrift`, quarry `docs/glyph.md` §5.

**Issue:** `CanonicalizeHandles` compares a re-sign's `Name` answer with the target and discards it.
`DoneChecks` judges only Create, Delete and Rename groups.
Quarry reports a C# parameter change only at the evidence tier, which `DetectDrift` never repairs and excludes from its deleted sweep.
For example, card 2 re-signs `Ns#R.Draw(int)` to `(double)` and pending card 3 uses `Ns#R.Draw(int)`.
Record-batch reports nothing, and card 3's begin-batch `resolvePass` is blocking `glyph-not-found`.
D7's "the done check and drift expect it" claims a behaviour no Decision adds.

### [BLOCKING] C# Rename derivation is still Go string work

**Location:** `_lyx/discussion/decision-record.md` D7 Rename bullet; evidence `internal/planparser/handle.go:113-128`, `internal/planglyph/handle.go:60-76,128-158`.

**Issue:** The new identifier comes from `HandleIdentifier`, which splits at the last `.`.
For `plan:Ns#R.Paint(System.IO.Stream)` that yields `Stream)`, and for `plan:Ns#R.Paint(int)` it yields `Paint(int)`.
D7 strips the arity suffix only on the old side, so `Box<>` renamed to `Crate<>` gives `public class Crate<><T>`.
The `KindConst` branch synthesizes the Go head `const X = 0`, which C# rejects.
`renameSignature` swaps the first whole-word match, which in `public Color Color { get; }` or in an attribute literal is the wrong token.
D7's "never from splitting the member at its last `.`" covers only Create handles.

### [MEDIUM] D4 misses a C# plan whose glyphs the Go alphabet also parses

**Location:** `_lyx/discussion/decision-record.md` D4; evidence `internal/planparser/validate.go:479-503`, quarry `docs/glyph.md` §5.

**Issue:** `Norce.Models#Pump`, `Norce.Models#Pump.Rate` and `Norce.Models#` all parse under the Go alphabet, so `glyph-malformed` never fires, and with no path targets neither does `plan-language-mismatch`.
Quarry's resolve answers `found` from C#, and the Go-toolchain checks then run on C# answers.
A Create handle is named with `Lang: go` and fails `handle-name-failed` with no hint at `language: csharp`.
The Rationale claims D4 covers cards that name only member glyphs, and it does not.

### [MEDIUM] A namespace pair passes as a file pair

**Location:** `_lyx/discussion/decision-record.md` D3 and Scope Out; evidence `internal/planparser/validate.go` `isFileRenamePair`.

**Issue:** Under `csharp`, `` `Norce.Models#` -> `Norce.Core#` `` is two C# self glyphs, so D3 makes it a file pair.
`path-missing` skips slash-free self glyphs and resolve finds the namespace, so the plan passes and the implementer runs `git mv` on a path that may not exist.
Scope puts namespace renames out of scope, but nothing refuses them.

### [MEDIUM] D3's bound omits checks and a site the helper cannot reach

**Location:** `_lyx/discussion/decision-record.md` D3; evidence `internal/planglyph/plangate.go:63`, `internal/batcher/boundary.go:32`, `internal/planparser/targetdirs.go:192-316`, `internal/planglyph/scope.go:164-188`.

**Issue:** The bound says "What that lets through, all of it", but it omits two losses:
- `redundant-file-target` skips `Program.cs#` read as C#.
- `RefCovers` maps no C# member or namespace glyph to a scope, so a later Delete of `src/Foo.cs#` draws no batch boundary before an earlier card's C# member target in that file.

`ScopeGuard` calls `RefMember(nil, …)` and `RefScope(nil, …)`, which read the default Go alphabet, so "every site goes through it" does not hold for it.

### [MEDIUM] D4's Go reading contradicts D3 and the Constraints

**Location:** `_lyx/discussion/decision-record.md` D3, D4, Constraints.

**Issue:** D3 says no caller parses with a language of its own, and the Constraints say a C# glyph is never read as a path.
D4 reads `Program.cs#`, which D3 reads as a C# namespace, in its Go-alphabet path reading.
The record does not say where that second reading lives, so either D4 breaks D3's rule or it parses outside `planparser`, against the declared PATTERN text.

### [MEDIUM] The relaxed re-sign guard has no bound

**Location:** `_lyx/discussion/decision-record.md` D7 re-sign bullet.

**Issue:** The guard ignores the parameter list, so it accepts a re-sign of `Draw(int)` to `Draw(double)` when `Draw(double)` already exists, which is a compile error or an `ambiguous` glyph.
It also accepts a head that drops the parentheses, a method re-signed into a property.
The rubric's attack-surface rule requires the Decision to state what the loosened guard lets through and what bounds it, and it states neither.

### [LOW] Explicit interface members pass a wrong Owner

**Location:** `_lyx/discussion/decision-record.md` D7 Create-handle bullet; evidence quarry `internal/engine/name.go:333-352,399-414`.

**Issue:** For `Renderer.System.IDisposable.Dispose()`, D3's owner chain carries the interface qualifier.
Quarry builds one class shell per Owner component and accepts extra owner components only when the head spells them, so `Name` fails `invalid_container`.

### [LOW] The listing rule governs no reader

**Location:** `_lyx/discussion/decision-record.md` D6 third bullet and the README.md acceptance criterion.

**Issue:** A grep for `Listing` over `internal`, `tools` and `cmd` finds only a comment at `internal/planglyph/handle.go:116`.
The bullet and the criterion point at a consumer that does not exist; this is an exclusion finding.

### [LOW] D1's re-sign Lang source contradicts D3 and D7

**Location:** `_lyx/discussion/decision-record.md` D1; evidence `internal/planglyph/handle.go:228-243`.

**Issue:** The re-sign declaration is built from `glyph.Parse` of the target, with no resolve answer involved.
D1 says the resolved glyph's language, and D3 and D7 say the parse.

### [LOW] D5 and D6 state two partitioning rules

**Location:** `_lyx/discussion/decision-record.md` D5, D6; evidence `internal/planglyph/buildenv.go:78-150`.

**Issue:** D5 gates partitioning on the plan's language, and D6 gates it on each candidate's language.
`filenameConstraintSuffix` would read `Foo_windows.cs` as a GOOS constraint in a mixed candidate set.

### [LOW] D5's rationale is false for the verify-package checks

**Location:** `_lyx/discussion/decision-record.md` D5; evidence `internal/planparser/verifypackage.go:26`.

**Issue:** `verify-package-missing` reads Verify command strings, including the language-neutral `lyx gate test`, not `.go` files.
Skipping it under `csharp` leaves `lyx gate test ./missing` unflagged, and the Bound does not say so.

### [LOW] A requirement sits in the optional Notes section

**Location:** `_lyx/discussion/decision-record.md` Notes for the plan writer.

**Issue:** The pattern background update must land, yet the spec makes Notes "never a completeness requirement", so the plan writer may drop it.
Extract it into Scope before cutting it from Notes.

### [NIT] Scope paragraph and an Open risk repeat Decisions

**Location:** `_lyx/discussion/decision-record.md` Scope and Open risks.

**Issue:** The "per file, by extension" paragraph restates D2, D3 and D4, and the support log's Turn 2 already records the departure.
The last Open risk restates D3's Bound.
Neither carries a requirement absent elsewhere, so both are relocation-and-cut items.

## Rejected

- D6's Kind switches (every switch accepts every new value): no exhaustive Kind switch exists. The `KindVar`/`KindConst` switch's C# defect is kept under F3, not as a Kind finding.
- The `PATTERN-refusal-way-forward` constraint binding only webster, shed and loom packages: citing it for plan findings is stricter than required and harms nothing.
- The C# fixtures appearing in Go-hub `lyx quarry` listings: already stated in Open risks, and not a defect.
- Missing call-site enumerations for the D3 helper sweep: the rubric's Do-not-flag list excludes them. F6 is kept only because D3 states its bound as complete, and it is not.
