---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: handler
    location: _lyx/plan/09-csharp-create-handles.md:24
    summary: >-
      Card 9's declaredNameIndex fallback (first identifier followed by one of the listed tokens) picks a generic type or interface-qualifier token,
      so card 9's own draft row IProvider<int>.draft cannot bind and card 10's renameCSharpDecl derives a head for a finalizer or static constructor instead of reporting false.
    fix: >-
      In card 9 (09-csharp-create-handles.md, the declaredNameIndex paragraph at line 24) replace the fallback rule with the declarator-position rule:
      with an empty name, or a name not found, the declared name is the last identifier at nesting depth zero, outside attribute lists and outside every (), [] and <> pair,
      before the first depth-zero (, [, {, =, ;, =>, the word where, or the end of the head; it reports false when that identifier is preceded by ~ or by the word static with no type before it.
      Keep the example sentence (int IProvider<int>.Count { get; } gives Count), which the new rule satisfies, and add class Box<T> gives Box and public void Wrap<U>(T item) where U : class gives Wrap as examples.
      In card 10 (10-csharp-rename-derivation.md, the renameCSharpDecl bullets at line 35) state that renameCSharpDecl calls declaredNameIndex with the resolved symbol's own name and never takes the fallback: a name not found as a whole identifier reports false,
      so a finalizer (~Renderer) and a static constructor (static Renderer) report false as the card claims, and add a finalizer signature row (~Renderer()) beside the operator row in TestRenameCSharpDecl, asserting false.
  - id: F2
    severity: BLOCKING
    class: design
    origin: handler
    location: _lyx/plan/09-csharp-create-handles.md:36
    summary: >-
      Card 9's row for plan:Glyphchain.Sharp#Renderer.IPair<string,int>.First with the spaced head int IPair<string, int>.First { get; } cannot bind clean,
      because quarry v0.4.0's checkMintedContainer compares the canonical qualifier IPair<string,int>. against the head text verbatim and answers invalid_container.
    fix: >-
      In card 9 (09-csharp-create-handles.md line 36) change the row's expectation to one handle-name-failed whose detail carries quarry's invalid_container,
      and add one sentence to the csharpOwner paragraph: csharpOwner compares in canonical spelling so the owner it hands quarry is Renderer,
      but quarry v0.4.0 (internal/engine/name.go checkMintedContainer) matches the qualifier against the head as written, so a head must spell the qualifier canonically (IPair<string,int>.) to bind.
      Keep the acceptance criterion's explicit-interface case covered by the System.IDisposable.Close() row and the repaired IProvider<int>.draft row.
  - id: F3
    severity: BLOCKING
    class: design
    origin: handler
    location: _lyx/plan/04-scope-guard-plan.md:26
    summary: >-
      Card 4's go row cannot report no finding, since under go the unit Glyphchain.Sharp holds a dot, so hasFileExtension makes RefScope a file scope that does not cover Glyphchain.Sharp/notes.txt;
      both rows report scope-outside-plan and the pair does not show the guard reads the plan's alphabet.
    fix: >-
      In card 4 (04-scope-guard-plan.md lines 26-28) replace the target Glyphchain.Sharp# with a dot-free namespace self glyph, Glyphchain#,
      and the changed file Glyphchain.Sharp/notes.txt with Glyphchain/notes.txt, so under go the target is the package directory Glyphchain covering the file (no finding)
      and under csharp it is a namespace with no scope (one scope-outside-plan).
  - id: F4
    severity: MEDIUM
    class: design
    origin: handler
    location: _lyx/plan/06-plan-language-mismatch.md:25
    summary: >-
      Card 6 counts too little as evidence of the plan's own language. Under go a Go package self glyph is neither a .go path nor a symbol answer, so a go plan deleting a package beside .cs fixtures is refused, though D4 reads any resolve answer naming a .go file.
      Under csharp a C# member target does not count, so a csharp plan editing C# members plus one .go file is refused, though D4's bound refuses only a plan whose only source targets are the other language's.
    fix: >-
      In card 6 (06-plan-language-mismatch.md): in the pure half (line 25), under go also skip the check when a Target is a self glyph whose PathEntryPath reading has no file extension, since such a package or directory may hold .go code and is the resolve half's to judge;
      under csharp, do not fire when some Target is a member glyph or a plan: handle, since under csharp such a ref names C# code.
      In the resolve half (line 31), count a found self glyph's Listing holding a .go file as an answer naming a .go file, beside a symbol in a .go file.
      Add TestValidate_LanguageMismatch rows: go with internal/x# and src/Foo.cs does not fire; csharp with Glyphchain.Sharp#Renderer.Title and tool.go does not fire;
      and a TestGlyphChain_CSharp go row editing Glyphchain.Sharp#Renderer.Title beside the package self glyph shapes# reports none.
  - id: F5
    severity: MEDIUM
    class: consistency
    origin: lens:affected-set
    location: _lyx/plan/03-planglyph-ref-alphabet.md:51
    summary: >-
      Card 3 removes the language value Plan.GlyphLanguage returned, yet its fallback for a Create handle body ParseRef rejects still needs glyph.CSharp under csharp, and no card names where planglyph gets it without a language mapping outside planparser.
    fix: >-
      In card 3 (03-planglyph-ref-alphabet.md line 51) name the source: the fallback takes Lang from the *glyph.ParseError that planparser.ParseRef returns for the rejected body (its Lang field, glyph.Go under go and glyph.CSharp under csharp, since ParseRef returns the plan alphabet's error),
      so no caller outside planparser maps a language string; with F6's fix applied the csharp branch makes no quarry call and the fallback runs only under go.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:way-forward
    location: _lyx/plan/03-planglyph-ref-alphabet.md:51
    summary: >-
      Under csharp a Create handle body ParseRef rejects, such as Renderer.Bad(Int32), surfaces only as handle-name-failed with quarry's owner-less naming error, which names no step the planner can take,
      while card 10 gives the Rename new side the C# spelling way forward.
    fix: >-
      In card 3 (03-planglyph-ref-alphabet.md line 51) state that under csharp a Create handle body ParseRef rejects is reported as handle-name-failed whose detail carries ParseRef's C# parse error and the way forward,
      spell the handle plan:<namespace>#<owner.member> in the C# alphabet with canonical parameter types, with no quarry call;
      change the Bad(Int32) row of TestGlyphChain_CSharp (line 61) to assert that detail names the C# handle spelling.
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:way-forward
    location: _lyx/plan/11-csharp-resign-arrows.md:35
    summary: >-
      Card 11 sends the parameter-list way forward (keep the member's parameter list, or change its kind through a Delete and a Create) on every C# resign-head-mismatch,
      including a head naming another member and a head quarry cannot name, where neither step clears the finding; D7 prescribes it only for a minted id that drops or adds the parameter list.
    fix: >-
      In card 11 (11-csharp-resign-arrows.md line 35) keep that detail for the parameter-list case only; for a C# head minting another member, name the way forward as a Rename pair for a name change or a head that declares the target member;
      for a naming error, name it as correcting the head to one quarry names under the owner chain, quoting quarry's error.
      Add a detail assertion to the another-member row and the fails-to-name row of TestGlyphChain_ResignScenarios.
  - id: F8
    severity: MEDIUM
    class: design
    origin: lens:way-forward
    location: _lyx/plan/08-multi-alphabet-answers.md:22
    summary: >-
      Card 8's cross-alphabet detail tells the planner to name the file, such as Glyphchain.Sharp/Foo.cs#, a path built from the namespace spelling that need not exist (the fixture's file is csharp/Renderer.cs);
      rendering it from the answer would read Glyph.Unit as a disk path, which PATTERN-glyph-conversion-chokepoint forbids.
    fix: >-
      In card 8 (08-multi-alphabet-answers.md line 22) make the detail name no constructed path:
      the languages from Languages, and the way forward, name the .cs file that declares it as a slashed path self glyph, or a member, as lyx quarry glyphs lists them;
      keep the test assertion on csharp, go and name the file.
  - id: F9
    severity: LOW
    class: scope
    origin: lens:guard-reach
    location: _lyx/plan/05-go-only-checks.md:33
    summary: >-
      Card 5 gates goPackageDir, the external-test unit mapping D5 confines to go, at RefScope and targetDir only; brokenFabricVocabularyRule (internal/planparser/vocabname.go:54) still trims a _test suffix from a csharp handle's unit.
    fix: >-
      In card 5 (05-go-only-checks.md) add the Edit target internal/planparser#brokenFabricVocabularyRule and extend the bullet at line 33: brokenFabricVocabularyRule maps the unit through goPackageDir only under go
      (CheckFabricVocabularyName passes whether the plan is go); add a csharp row to TestCheckFabricVocabularyName whose handle unit ends _test and whose dir is used unchanged.
      Checked with lyx loom validate-plan --overlay: no new finding.
  - id: F10
    severity: LOW
    class: design
    origin: lens:guard-reach
    location: _lyx/plan/05-go-only-checks.md:42
    summary: >-
      Card 5's only caller-coverage row asserts no caller-uncovered under csharp for a Delete of the C# Renderer.Title, with no go counterpart showing the finding fires without the gate, so the row may pass with the callerCoverageFindings gate removed.
    fix: >-
      In card 5 (05-go-only-checks.md line 42) make the row a go and csharp pair over one Go-side subject that caller coverage reports under go:
      card 1 deletes the slashed Go member nested/inner#Help, which the Go alphabet parses under csharp too, and nested/user/user.go calls inner.Help();
      the go row reports caller-uncovered, the csharp row none.
  - id: F11
    severity: LOW
    class: consistency
    origin: lens:way-forward
    location: _lyx/plan/12-resign-post-batch.md:29
    summary: >-
      Card 12 builds the resign-not-done detail in internal/planglyph, which has no batch number, so it names lyx webster recover-batch NN with a placeholder, where deleteReferencedWayForward fills the real number.
    fix: >-
      In card 12 (12-resign-post-batch.md line 29) let the resign-not-done detail name the card and the edit or head correction with lyx webster rebaseline --card <card number>,
      and state that the batch's recover-batch step comes from websterengine's not-done way forward, which knows the batch number.
  - id: F12
    severity: LOW
    class: design
    origin: lens:way-forward
    location: _lyx/plan/07-rename-namespace-pair.md:18
    summary: >-
      Card 7's rename-namespace-pair detail names no step for a root-file old side that a later card creates, since the check exempts only an earlier card's Create target or Rename destination.
    fix: >-
      In card 7 (07-rename-namespace-pair.md line 18) add a third way forward to the detail: when a later card creates the old side, move the pair to a card after it.
  - id: F13
    severity: NIT
    class: consistency
    origin: lens:affected-set
    location: _lyx/plan/13-batcher-resign-boundary.md:13
    summary: >-
      Card 13 adds the C# re-sign boundary, but internal/batcher/boundary.go's file header still says boundaries come only from a later card's rename or delete, and no card rewords it.
    fix: >-
      In card 13 (13-batcher-resign-boundary.md, Intent) add one sentence: boundary.go's file header names the re-sign boundary beside the rename-and-delete cover rule;
      list no file self glyph for it, since redundant-file-target refuses a file beside a member declared in it.
---

### [BLOCKING] declaredNameIndex fallback picks the wrong token

**Location:** `_lyx/plan/09-csharp-create-handles.md:24`, `_lyx/plan/10-csharp-rename-derivation.md:35`

**Issue:** The fallback is the first identifier followed by `(`, `<`, `{`, `=`, `;`, `:`, `where` or the end.
In card 9's own row head `int IProvider<int>.Count { get; }` that identifier is `IProvider` (followed by `<`), not `Count`.
So `csharpOwner` finds no qualifier before it and keeps the whole chain `Renderer.IProvider<int>`.
Quarry v0.4.0 refuses an owner component with type arguments (`internal/engine/name.go` `nameCSharpContainer`, lines 340-345) with `invalid_container`, so the row ends in `handle-name-failed`, not the clean bind it asserts.
The card's example sentence contradicts its own rule.
The same fallback picks `List` in a draft head `public List<int> Get()`.
Card 10 reuses `declaredNameIndex`: a finalizer's name `~Renderer` and a static constructor's `static Renderer` are not whole identifiers, so the fallback lands on `Renderer` (followed by `(`), and a head such as `~NewName()` goes to `Name` instead of the `false` card 10 promises.
Fork guard-reach reported the same defect independently.

### [BLOCKING] Spaced interface qualifier cannot bind through quarry v0.4.0

**Location:** `_lyx/plan/09-csharp-create-handles.md:36`

**Issue:** Quarry mints the explicit-interface qualifier canonicalized (`internal/engine/csharp.go:371`, `csharpExplicitOwner`), giving owner components `Renderer`, `IPair<string,int>`.
`checkMintedContainer` (`internal/engine/name.go:410-413`) then accepts the extra component only when `strings.Contains(d.Decl, "IPair<string,int>.")`.
The head `int IPair<string, int>.First { get; }` does not hold that text, so quarry answers `invalid_container` and the row is `handle-name-failed`.
`csharpOwner`'s canonical comparison runs before quarry's check and cannot rescue it.
D7 only claims that `csharpOwner` matches the spaced spelling, which stays true; the row's end-to-end expectation is what is wrong.

### [BLOCKING] Card 4's go row reports the finding it claims is absent

**Location:** `_lyx/plan/04-scope-guard-plan.md:26`

**Issue:** Under `go`, `Glyphchain.Sharp#` parses with unit `Glyphchain.Sharp`.
`hasFileExtension` (`internal/planparser/normalize.go:124-130`) reads any dot in the last segment as an extension, so `RefScope` (`internal/planparser/targetdirs.go:177-178`) returns the file scope `Glyphchain.Sharp`.
`targetsCover` (`internal/planglyph/scope.go:192`) then compares it with `Glyphchain.Sharp/notes.txt` and does not cover it.
Both rows report `scope-outside-plan`, so the pair cannot fail when the guard still reads the default Go alphabet, which is the one thing the card says the `csharp` row proves.
Fork guard-reach reported the same defect independently.

### [MEDIUM] plan-language-mismatch counts too little as own-language evidence

**Location:** `_lyx/plan/06-plan-language-mismatch.md:25`, `_lyx/plan/06-plan-language-mismatch.md:31`

**Issue:** D4's resolve half fires "when neither a path target nor a resolve answer names a `.go` file".
Card 6 narrows that to "no found or multipart answer has a symbol in a `.go` file".
A found Go package self glyph answers with a `Listing` and no `Symbols` (`internal/engine/answer.go`, `ResolveResult.Listing`).
Its `PathEntryPath` reading is the directory `internal/x`, with no `.go` extension, and the pure half's member-glyph exemption does not cover a self glyph.
So a `go` plan that deletes or edits a Go package `internal/x#` beside `.cs` fixtures is refused by the pure half, and by the resolve half too when it also names a `.cs` member.
Under `csharp`, a member glyph is necessarily C#, yet the pure half fires for a plan with C# member targets plus one `.go` path.
D4's bound refuses only "a plan whose only source targets are the other language's files".

### [MEDIUM] Card 3's fallback has no named source for its Lang

**Location:** `_lyx/plan/03-planglyph-ref-alphabet.md:51`

**Issue:** Card 3 makes `Plan.GlyphLanguage` return only a bool, and the Shared Decisions say no caller outside planparser receives a language value.
Yet the fallback for a rejected Create body sets `Lang` to `glyph.Go` under `go` and `glyph.CSharp` under `csharp`, and card 3's own `Bad(Int32)` row reaches it.
The card leaves the implementer two options.
One is a new planparser accessor no card creates.
The other is a local mapping of `plan.Language` in planglyph, which goes against the constraint that the alphabet choice lives in planparser.

### [MEDIUM] A rejected C# Create handle body gets no actionable way forward

**Location:** `_lyx/plan/03-planglyph-ref-alphabet.md:51`

**Issue:** The fallback hands quarry an owner-less C# head.
Quarry's error ends "a C# member head needs Owner (CLI --owner)" (`internal/engine/name.go:322-328`), and a plan has no Owner field.
`checkHandleMalformed` (`internal/planparser/validate.go:605-655`) checks only for a `#`, so nothing else tells the planner that `Int32` must be spelled `int`.
That spelling is quarry's `ReasonNonCanonicalType` in `checkCSharpParams`.
Card 10 gives the Rename new side the C# spelling way forward for the same failure, so the two handle kinds disagree.

### [MEDIUM] C# resign-head-mismatch way forward misapplied

**Location:** `_lyx/plan/11-csharp-resign-arrows.md:35`

**Issue:** For `Draw(int) -> public void Other(int count)` the fix is a Rename pair, not a kept parameter list.
For `public void Draw(` or an `invalid_container` answer, the fix is a head quarry can name.
The single C# detail sends the planner to the parameter list in all three cases.
The another-member row and the fails-to-name row assert no detail, so nothing pins the wording.

### [MEDIUM] Cross-alphabet detail names a constructed file path

**Location:** `_lyx/plan/08-multi-alphabet-answers.md:22`

**Issue:** A cross-alphabet `ambiguous` answer carries no candidates (`internal/engine/answer.go:262-263`), so the detail cannot know which file declares the namespace.
In card 8's own fixture that file is `csharp/Renderer.cs`, and following the example `Glyphchain.Sharp/Foo.cs#` yields `path-missing`.
Building `<unit>/Foo.cs#` from the answer would read `Glyph.Unit` as a disk path, which `pattern/PATTERN-glyph-conversion-chokepoint.md` forbids.

### [LOW] goPackageDir still runs unguarded in the fabric-vocabulary check

**Location:** `_lyx/plan/05-go-only-checks.md:33`

**Issue:** `brokenFabricVocabularyRule` calls `goPackageDir(unit)` (`internal/planparser/vocabname.go:54`) under every language.
D5 lists the external-test unit mapping among the checks skipped under `csharp`, and card 5 gates only `RefScope` and `targetDir`.
Card 2 edits `CheckFabricVocabularyName` for the declared name but leaves this call alone.

### [LOW] Caller-coverage gate has no discriminating row

**Location:** `_lyx/plan/05-go-only-checks.md:42`

**Issue:** The `nested/inner` pair discriminates only the `laterImportFindings` half of the gate.
The caller-coverage row has no `go` counterpart.
A C# member subject is matched in Go code by name alone, which `callercoverage.go` reports informationally at most.
So the row may pass with the `callerCoverageFindings` gate removed.

### [LOW] resign-not-done names a placeholder batch number

**Location:** `_lyx/plan/12-resign-post-batch.md:29`

**Issue:** `RebindResigns` lives in `internal/planglyph` and cannot know the batch number.
`deleteReferencedWayForward` (`internal/websterengine/recordbatch.go:87-92`) prints the real number with `%02d`.
The step still works, but the operator has to look the number up.

### [LOW] rename-namespace-pair misses the move-after-create step

**Location:** `_lyx/plan/07-rename-namespace-pair.md:18`

**Issue:** A root-file old side that a later card creates is refused, since only an earlier card's Create target or Rename destination exempts it.
The detail names only a namespace rename through Edit cards, or pointing at an existing file.
The step that works, moving the pair after the creating card, is not named.

### [NIT] boundary.go's file header goes stale

**Location:** `_lyx/plan/13-batcher-resign-boundary.md:13`

**Issue:** `internal/batcher/boundary.go:1` describes boundaries from a later card's rename or delete only.
Card 13 lists only `renameBoundaries`, and card 14 covers `internal/batcher/doc.go`, so no card updates that header.

## Rejected

- guard-reach: card 5's `TestRefCovers` `csharp` row (`src/Foo.cs` versus `Glyphchain.Sharp#Renderer.Title`) passes because of card 2's `ParseRef`, not card 5's gate. That is no defect: the row pins D3's bound that a C# member maps to no scope, whichever card adds it.

## Focus verdicts

- D7 fidelity across cards 9-13: each sub-rule has a carrying card. Owner without the interface qualifier is card 9; the minted-id acceptance rule and the three `resign-overload-collision` variants are card 11; `resign-not-done`, the plan-wide rewrite and the `restampFingerprint` order are card 12; the batch boundary is card 13. No card adds unlicensed scope. Defects are F1, F2 and F7.
- D3 and D4 bounds: card 7 carries the disk-reading, pending-only `rename-namespace-pair`. Card 6 carries both mismatch halves, and every named input is reached by a named test. The evidence gap is F4.
- Attack surface (cards 6, 7, 11, 12): every new refusal or finding states what it lets through and what bounds it. No upstream check rejects a named test input before it reaches its refusal. Card 3's `Bad(Int32)` passes `checkHandleMalformed`, and a Go-parsed C# member skips `path-missing`.
- Card order: `ParseRef` (card 2), `Plan.GoToolchain` (card 5), the plan-aware `ScopeGuard` (card 4) and `nameDeclarations` (card 11) are each introduced before every card that uses them.
- Edit glyph hygiene on cards 2, 3, 5 and 11: member glyphs throughout, and re-sign arrows on every re-signed member, including card 4's interface method `planindex#Delta.ScopeGuard`, which correctly takes none.
- Test economy: the new rows extend existing tables. `TestParseRef` replaces `TestParseGlyph`. `TestRenameCSharpDecl` and `TestRebindResigns` pin exact heads and per-arrow outcomes that no glyph-chain row observes. No duplicate found.
- Research steps: no card leaves a quarry v0.4.0 fact to be found at run time. Two of the facts card 9 asserts are wrong (F1, F2).
- Verify lines and PATTERN fit: no card carries a `**Verify:**` line, so no raw test command. Card 14 writes the declared `PATTERN-glyph-conversion-chokepoint` text verbatim. The only chokepoint risk found is F8.
