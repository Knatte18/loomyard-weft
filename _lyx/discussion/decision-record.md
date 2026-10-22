# Decision record — quarry-csharp

## Goal

LoomYard takes quarry `v0.4.0` and plans, reviews and implements with C# glyphs in a C# hub such as NORCE Models, with every Go plan behaving as today.

## Scope

In:

- Raise `github.com/Knatte18/quarry` in `go.mod` from `v0.2.1` to `v0.4.0`, taking in `v0.3.0`'s Enclose unused, and fix the library break: every `quarry.Declaration` sets `Lang`.
- A plan's `language:` key gains the value `csharp`, beside `go` (still the default) and `none`.
- The rule that picks the alphabet for each plan ref and handle, the extension guard on the plan's language, and the C# side of `plan:` handles, re-sign arrows and Renames.
- Reading quarry `v0.4.0`'s multi-alphabet answers: cross-alphabet ambiguity and the new Kind values.
- The planner stencil, Plan-Review rubric and the plan spec describe the C# alphabet; the package docs of `internal/planparser` and `internal/planglyph` describe the new rules.
- The pattern background file `pattern/PATTERN-glyph-conversion-chokepoint.md` gains the alphabet rule and the rule that path self glyphs use the Go unit alphabet under every glyph-enabled language.

Out:

- Wiring Enclose into any check: board entry `enclose-checks` owns that, and this raise is the `go.mod` bump that entry names.
- C# counterparts of the Go-toolchain checks (caller coverage, delete-order import scan, build-constraint partitioning, verify-package checks): no board entry asks for them.
- A plan that uses glyphs for Go and C# code at once; such a plan sets `language: none`.
- Renaming a C# namespace through a `Rename` pair.
- .NET verify gates (`dotnet-scoped-verify`, `dotnet-verify-protocol`), language lens sets (`lenses-language-sets`), the kick-start pack's removal (`kickstart-pack-remove`).

## Decisions

### D1 — Raise quarry to v0.4.0 and set `Lang` on every declaration

Decision: `go.mod` pins quarry `v0.4.0`.
Every `quarry.Declaration` LoomYard builds sets `Lang`: the plan's language for a Create handle, the language of D3's parse of the arrow's target for a re-sign arrow, and the resolved glyph's language for a Rename's derived declaration.
A C# declaration also sets `Owner` (D7).
No call site leaves `Lang` zero.

Rationale: quarry rejects a zero `Lang` per answer, so a missed site turns every handle binding into `handle-name-failed`.

### D2 — `language: csharp` names the C# alphabet for a plan

Decision: the overview's `language:` key accepts `go`, `csharp` and `none`, and an absent key still means `go`.
`plan-language-unrecognized` accepts `csharp`.
The per-language source-extension table that `ReviewExempt` reads maps `csharp` to `.cs`.
The planner sets `language: csharp` when the code its cards change is C#, read from the extension of the files it plans.

Rationale: the plan names the alphabet once, by the files' extension; keeping `go` as the default leaves every existing plan unchanged.

### D3 — One planparser helper picks each ref's alphabet

Decision: `internal/planparser` exports one helper that, given a plan and a `#`-shaped ref or a `plan:` handle's body, returns its parsed glyph.
Every site in `internal/planparser`, `internal/planglyph` and `tools/plandag` that parses a plan ref or splits a handle's member goes through it.
`ScopeGuard` receives the plan, so its `RefMember` and `RefScope` calls go through the helper rather than reading the default Go alphabet.
`Plan.GlyphLanguage` keeps only its glyph-enabled answer; no caller parses with a language value of its own.
Under `go` the helper parses with `glyph.Go`, exactly as today.
Under `csharp`:

- a ref the C# alphabet parses is a C# glyph, including a ref both alphabets parse;
- a ref only the Go alphabet parses is accepted only when it is a self glyph: a path self glyph such as `src/Foo.cs#`;
- any other ref is `glyph-malformed`, with the C# alphabet's parse error.

Path-shaped entries canonicalize through `glyph.Self(glyph.Go, path)` under both `go` and `csharp`, because quarry spells a path only in the Go unit alphabet.
`internal/planparser` also exports the Go-alphabet path reading of a canonicalized path entry; D4 and the namespace-pair check below are its only callers.

A rename pair whose two sides are both self glyphs is a file pair under `csharp`, so `Program.cs -> App.cs` still runs `git mv`, as under `go`; the pair classification reads no disk.
The plan gate refuses a pending pair whose old side is a slash-free self glyph whose Go-alphabet path reading names nothing on disk and nothing an earlier card creates or renames into, with the new blocking finding `rename-namespace-pair`.
Such a pair names a C# namespace, such as `Norce.Models# -> Norce.Core#`, and the finding's detail names the way forward: rename the namespace through Edit cards on the files that declare it and the files that refer to it.
The check reads the disk rather than the extension, since a dotted namespace such as `Norce.Models` has a path extension too, and it runs only on pending cards, so a pair whose `git mv` has landed is never rechecked.

Bound: under `csharp`, a slash-free self glyph such as `README.md#`, `Program.cs#` or `docs#` parses as a C# namespace, and so has no `UnitPath`.
What that lets through, all of it:

- `card-path-malformed` and `path-missing` skip it; its existence is checked only by the resolve pass, where quarry tries both alphabets.
- `ReviewExempt` counts it as a source target, so such a plan goes to Plan-Review.
- The batcher's card estimate and the target-directory derivation count no file for it, nor for any C# member or namespace glyph; a C# card is costed by its slashed path targets only.
- The redundancy check sees it as a namespace, so it raises no package-versus-member redundancy for it.
- `redundant-file-target` is not raised for it, since it lists no file.
- `planparser.RefCovers` maps no C# member or namespace glyph to a scope, so a later Delete or Rename of a file holding an earlier card's C# member target draws no batch boundary.

A slashed path self glyph keeps every path check.
A slash-free self glyph that names nothing yet, such as the Create target `Tool.cs#`, answers `not_found` with `unit: not_found`, per quarry `v0.4.0` `docs/glyph.md` §5 (if none matches, `not_found`, with `unit: found` when any accepting alphabet found the unit).
The Create check passes that answer with at most the informational `create-new-unit`, exactly as for a missing Go file today.

Rationale: quarry's own resolve tries every alphabet, but LoomYard's checks need one parsed glyph per ref; reading the overlap as C# costs only the bounded items above, and one helper keeps the choice in the package that owns ref shapes.

### D4 — The plan's language is checked against its files' extensions

Decision: a new blocking plan finding, `plan-language-mismatch`, fires when a plan's path targets name source files of the other glyph language (`.cs` under `go`, `.go` under `csharp`) and none of its own language.
It reads the canonicalized path entries through D3's exported Go-alphabet path reading, so a root `tool.go` counts under `csharp`, and this half reads no disk.
Under `go` it also fires in the plan gate's resolve pass, when neither a path target nor a resolve answer names a `.go` file and the batched resolve answers a plan target with a symbol declared in a `.cs` file.
Its detail names the way forward: set `language:` to the language of the files, or `language: none` for a plan that changes both.
It does not fire under `none`.

Under `go`, a `glyph-malformed` ref that the C# alphabet parses gets a detail naming `language: csharp` as the way forward.

Bound: a Go plan that edits `.cs` fixtures beside any `.go` file is not refused; a plan whose only source targets are the other language's files is, and so is a `go` plan whose member glyphs resolve into `.cs` files and none into a `.go` file, with no `.go` path target.
A `go` plan whose only C# refs are namespace self glyphs, such as `Norce.Models#`, is not caught by this finding: on any ref `path-missing` checks, it refuses it when nothing exists on disk at that path, file or directory, and no card, earlier or later, names that exact path as a Create target or Rename destination.
Resolve answers `glyph-ambiguous` when a same-named directory exists and a file declares the namespace, and `found` when no file declares it.
Neither finding's detail names `language: csharp`.

Rationale: a C# hub's planner that leaves the default would otherwise meet findings that do not say what is wrong: `glyph-malformed` for a member glyph only the C# alphabet parses, and `handle-name-failed` for a plan whose glyphs the Go alphabet also parses.

### D5 — Go-toolchain checks run only under `language: go`

Decision: the checks that read Go source or Go toolchain semantics run only when the plan's language is `go`: caller coverage (`go/types`), the delete-order import scan, the verify-package checks, the `.go` file-unit handle rule and the external-test unit mapping (`goPackageDir`).
Under `csharp` they are skipped.
Build-constraint partitioning keys on its candidates' files, not on the plan's language (D6).

Bound: a C# plan gets no caller-coverage, import-order or verify-package finding, so a C# card's Verify package arguments are not checked, including those of the language-neutral `lyx gate test`; every glyph check that rests on quarry's answers (resolve, Create, done, scope, drift, delta) still runs.

Rationale: caller coverage, the import scan, the `.go` handle rule and the test-unit mapping key on `.go` files and the Go toolchain and have no meaning for a `.cs` file.
The verify-package checks read Verify command strings instead, and what a package argument names in a C# hub is the .NET verify gates' question, out of this task's scope.

### D6 — LoomYard reads quarry v0.4.0's multi-alphabet answers

Decision:

- A cross-alphabet `ambiguous` answer for a self glyph, with no candidates and a `languages` list, yields `glyph-ambiguous` whose detail names the languages and quarry's way out: name the file or a member.
- A member glyph's cross-alphabet `ambiguous` answer is handled like any other ambiguous answer.
- `classifyAmbiguity` partitions only a candidate set whose files are all `.go`, under every plan language; a candidate set holding a `.cs` file is never partitioned, so a `.cs` file name such as `Foo_windows.cs` is never read as a build constraint.
- Every switch over quarry's Kind accepts every value `v0.4.0` adds.

Rationale: these are the answer shapes `v0.4.0` adds; a reader written for `v0.2.1` would treat an empty candidate list as an error.

### D7 — `plan:` handles, re-sign arrows and Renames under `csharp`

Decision:

- A C# handle is `plan:<namespace>#<owner.member>` with a C# declaration head.
  Its namespace, owner chain and name come from D3's helper parsing the handle's body with the C# alphabet, never from splitting the member at its last `.`.
  Binding hands quarry's `Name` a declaration with `Lang: csharp`, the namespace as `Unit`, the owner chain with the interface qualifier removed, per the next bullet, as `Owner` and the head as `Decl`.
- Every C# declaration LoomYard hands `Name`, for a handle, a re-sign arrow or a Rename, sets `Owner` to its glyph's owner chain with the member's interface qualifier removed.
  The qualifier is the longest trailing run of owner-chain components that the head writes, dotted and followed by a dot, immediately before the declared member name, compared in quarry's canonical C# spelling, so `IProvider<string, int>.` in a head matches the component `IProvider<string,int>`.
  The declared member name is the occurrence the Rename bullet's declared-token rule picks, so in `public Item.Value Value { get; }` it is the second `Value`.
  A head that does not write them there leaves the chain whole, as for a member of a nested type.
  So `void System.IDisposable.Dispose()` for the glyph `Renderer.System.IDisposable.Dispose()`, and `int IProvider<A>.Value { get; }` for `Renderer.IProvider<A>.Value`, both have the owner `Renderer`:
  quarry spells the qualifier inside the owner chain, refuses an owner component with type arguments, and reads the head's qualifier again.
- A C# re-sign arrow's declaration takes its `Unit` and owner chain from D3's parse of the arrow's target, and its head from the arrow.
  The arrow is accepted when the id `Name` mints equals the target glyph apart from its parameter list, since C# parameter types are part of the glyph.
  Bound: the minted id keeps the target's parameter-list presence, both with parentheses or both without, so a method is never re-signed into a property;
  a minted id that drops or adds the parameter list is `resign-head-mismatch`, whose C# detail names the way forward: keep the member's parameter list in the head, or change the member's kind through a Delete and a Create.
  The new blocking finding `resign-overload-collision` refuses a re-sign whose minted id differs from the target and already resolves found, unless the same or an earlier card deletes, renames away or re-signs away the member that id names.
  Its detail names the overload it collides with and the way forward: change the head to a free signature, or remove or re-sign that overload on an earlier card.
  It also refuses a minted id that equals the canonical id any card's Create handle or Rename new handle binds to, unless a card between the two declarations deletes, renames away or re-signs away the member the earlier one declares, and its detail then names the card that declares that handle and the way forward: change the head to a free signature, or drop one of the two declarations.
  It also refuses a minted id that equals the minted id of another re-sign arrow in the plan whose target is a different glyph, with the same exception for an intervening card that removes the member one of the two declares, and its detail then names the other arrow's card and the way forward: change one head to a free signature.
  The re-sign arrows' declarations are named before the plan's batched resolve, and each minted id that differs from its target joins that resolve's target set, where `not_found` is its expected answer and raises nothing.
- A later card names a C# member re-signed by an earlier card by its old glyph, which is what resolves when the plan is checked.
  The batcher starts a new batch at the first later card that names a C# re-sign arrow's target, so no batch holds both the re-sign and a later use of the member.
- When a C# re-sign's minted id differs from its target, the post-batch pass that records the re-sign's card names the arrow again and resolves the minted id.
  That pass is shared by record-batch and recover-batch, so a recovered batch gets the same check and rewrite.
  An answer other than `found` is the new blocking done check `resign-not-done`, whose detail names the way forward: make the edit match the arrow's head, or correct the head to the signature the card wrote.
  A found answer rewrites the old glyph to the minted id plan-wide through `planparser.RewriteRefs`, as handle binding does, so a later card naming the member resolves at its own dispatch.
  The rewrite runs after the done checks, either inside the `RewriteRefs` call that handle binding makes or in its own call, and in both cases is followed by `restampFingerprint` before the scope guard and `DetectDrift` run, so the next begin-batch meets no fingerprint mismatch.
  It also applies to the in-memory plan the pass hands `DetectDrift`, so no later card still names the old glyph there, and `ScopeGuard`'s target union for the re-sign card holds both the old glyph and the minted id.
- A C# Rename's derived declaration takes its `Unit` and owner chain from the resolved glyph and the head from the resolved symbol's signature.
  Its new identifier comes from D3's helper parsing the new handle's body, with the generic arity suffix stripped (`Crate<>` gives `Crate`), never from `planparser.HandleIdentifier`'s last-dot split.
  The substitution replaces the declared name token: the first occurrence of the old name, without its arity suffix, outside attribute lists, that is followed by `(`, `<`, `{`, `=`, `;`, `:`, `where` or the end of the head.
  So `class Box<T>` gives `class Crate<T>`, and `public Color Color { get; }` renames its second `Color`.
  Bound: a head whose own type is a generic of the member's name, such as `Color<int> Color { get; }`, has the type's token replaced.
  The var and const head synthesis applies only to Go symbols.
  A C# field, const or event derives a one-declarator head instead, since quarry gives every declarator of one declaration the whole declaration as its signature, so a substitution in `public int a, b;` declares two symbols and `Name` refuses it with `several_declarations`.
  The head is the resolved signature cut before its first declarator's name, followed by the new identifier, with ` = default` appended for a const.
  The declarator name is the last identifier at nesting depth zero, outside attribute lists and outside every `()`, `[]` and `<>` pair, before the first depth-zero `,`, `=` or `;`, or before the end of the signature.
  So `private Dictionary<string, int> map;` renamed to `renamed` gives `private Dictionary<string, int> renamed`, and `(int, int) pair;` keeps its tuple type.
  So `public const int A = 1, B = 2;` renamed from `A` to `C` gives `public const int C = default`, and an event with accessors, whose signature quarry cuts before them, gives an event field head that mints the renamed event's id.
  A type's constructors follow the type through the hub's rename tool, with no pair of their own.
- `isInterfaceMethod` answers false for every symbol whose glyph is C#, so `resign-interface-method` and the Rename's `func (Owner)` receiver synthesis stay Go-only.
- A C# symbol `Rename` needs a C# entry in the hub's `language.yaml` `rename_tools`; `rename-tool-missing` already names the way forward.

The Go handle, re-sign and Rename rules stay as they are under `go`.

Rationale: quarry `name` needs the owner for a C# member, since a C# head states no owner as a Go method states its receiver, and a C# member's spelling holds parameter lists and dotted types that a string split cuts in the wrong place.

### D8 — Planner, Plan-Review and the kick-start inherit C# glyphs; no new mechanism

Decision: the `lyx quarry` verbs pass C# answers through unchanged, since quarry walks `.cs` files itself.
The kick-start pack and every plan gate take their alphabet from D3, so they get C# glyphs once the plan says `csharp`.
The planner stencil and `contracts/specs/loom-plan-spec.md` describe `language: csharp`, that a C# member glyph is copied from a quarry answer with its parameter list, that a `.cs` file is written as a plain path, D4's finding, `rename-namespace-pair`, the C# re-sign bounds of D7, and that a later card names a re-signed C# member by its old glyph.
The Plan-Review rubric's rule that an `Edit` names Go file paths is worded for both languages.

Rationale: the board entry's three consumers already read glyphs only through quarry and the plan's alphabet, so the work is the alphabet choice and the words that teach it.

## Constraints

- `PATTERN-glyph-conversion-chokepoint`: `glyph.Self` stays the only path-to-glyph call and `Glyph.UnitPath` the only glyph-to-path call; D3 adds no local grammar, and a C# glyph is never read as a path, apart from the one Go-alphabet reading of a canonicalized path entry that `internal/planparser` exports (D3).
- `PATTERN-ref-shape-registry` and `PATTERN-sole-parsers`: the alphabet choice lives in `internal/planparser`, the plan's sole parser.
- `PATTERN-refusal-way-forward`: `plan-language-mismatch`, the `glyph-malformed` detail of D4, `rename-namespace-pair`, `resign-overload-collision`, `resign-not-done` and the cross-alphabet `glyph-ambiguous` detail name the way forward, and a test reaches each.
- `PATTERN-quarry-cgo`: the C# grammar binding links through cgo, which the build already needs.
- `PATTERN-stencil-ownership` and `PATTERN-producer-pointer-rule`: the planner stencil points at the plan spec for the C# rules rather than restating it.
- `PATTERN-test-economy`, `PATTERN-test-speed`, `PATTERN-test-tiers`: C# coverage extends the existing glyph tests with `.cs` fixtures, run through `lyx gate test`.
- `PATTERN-documentation-lifecycle`: the rules land in `internal/planparser/doc.go` and `internal/planglyph/doc.go`.

## Auto-mode assumptions

- The plan names its alphabet through `language:`, picked by the planner from the extension of the files it plans, rather than per ref or per hub.
- A ref both alphabets parse is read as C# under `csharp`.
- A plan that changes Go and C# code at once uses `language: none`.
- `plan-language-mismatch` is blocking, within D4's bound.
- The Go-toolchain checks get no C# counterpart in this task.

## Open risks

- Quarry's C# round trip over a real C# checkout has not been run, so a C# answer may miss a symbol; LoomYard reports it like any `not_found`.
- A repository where a directory and a namespace share a spelling makes self glyphs resolve `ambiguous`; the planner must then name a file or a member.
- The `.cs` fixtures this task adds show up in `lyx quarry` listings of their directories in this Go hub, and a resolve batch holding any target that parses as a C# glyph, such as a slash-free self glyph, now scans the repository's `.cs` files.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 3: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lyx/reviews/discussion/round-3-review.md; fixer report: _lyx/reviews/discussion/round-3-fixer-report.md.
  - explicit-interface-member-owner (class design, severity MEDIUM, open in rounds 1, 2, 3)
  - field-head-cut-ignores-generics-tuples (class design, severity MEDIUM, open in rounds 3)
  - resign-batch-order-vs-driftcheck-scopeguard (class design, severity MEDIUM, open in rounds 2, 3)
<!-- /lyx:carry-over Discussion-Review -->

## Acceptance criteria

- `go.mod` pins quarry `v0.4.0`, and every Go plan fixture yields the same findings as before.
- Binding a Go handle, a Go re-sign arrow and a Go Rename succeeds with `Lang` set.
- Under `csharp`, a Create handle whose member has a parameter list holding a dotted type binds to a C# glyph with the right owner and name, and an explicit interface member's handle binds with an `Owner` that leaves out the interface qualifier.
- Under `csharp`, a handle for a member of the nested type `Outer.Item` whose head `Item.Kind Get()` names a type of the same spelling binds with the full owner `Outer.Item`.
- Under `csharp`, after a C# re-sign batch is recorded or recovered, the next begin-batch is not refused with a fingerprint mismatch.
- Under `csharp`, a C# method re-sign that changes a parameter type raises no `resign-interface-method` and is accepted; once its card is recorded, a later card naming the old glyph names the minted id; a minted id that drops the parameter list is `resign-head-mismatch`, one that collides with an existing overload is `resign-overload-collision` unless an earlier card removes that overload, one that equals the id a Create handle or a Rename new handle elsewhere in the plan binds to is `resign-overload-collision` naming that handle's card, one that equals the id another re-sign arrow on a different target mints is `resign-overload-collision` naming that arrow's card while two re-signs of the same target are not, and an edit that leaves the minted id unresolved is `resign-not-done`.
- Under `csharp`, a re-sign arrow on an explicit interface member such as `Ns#Renderer.IProvider<A>.Value` mints its own target id.
- Under `csharp`, a C# Rename of a generic type, of a method, of a const, of `A` in a const declared as `public const int A = 1, B = 2`, and of a property whose type shares its name each derives its declaration, and a C# method Rename's head carries no `func` receiver.
- Under `csharp`, renaming a field declared as `private Dictionary<string, int> map;` to `renamed` derives the head `private Dictionary<string, int> renamed`, and a tuple-typed field derives a head that keeps its tuple type.
- Under `csharp`, a pair of two C# namespace self glyphs is `rename-namespace-pair`, and a root file pair whose old side an earlier card creates is not.
- Under `csharp`: a C# member glyph with a parameter list, a C# namespace self glyph and a path self glyph of a `.cs` file each parse and resolve; a ref neither alphabet takes is `glyph-malformed`.
- Under `csharp`, `README.md` as a path target canonicalizes to `README.md#`, skips the path checks and resolves `found` through quarry.
- Under `csharp`, the root file pair `Program.cs -> App.cs` is a file pair.
- Under `go`, a C# member glyph is `glyph-malformed` with a detail naming `language: csharp`.
- `plan-language-mismatch` fires for a plan whose only source targets are `.cs` under `go`, or `.go` (a root `tool.go` included) under `csharp`; it fires under `go` for a plan with no `.go` path target whose member glyphs resolve into `.cs` files and none into a `.go` file; it does not fire for a Go plan with `.go` and `.cs` targets, whether named as paths or as member glyphs, nor under `none`; it names the way forward.
- Under `csharp`, caller coverage, the delete-order import scan and the verify-package checks report nothing.
- A cross-alphabet ambiguous self glyph yields `glyph-ambiguous` naming both languages.
- `ReviewExempt` treats a `.cs` Prosa target as source under `csharp`.
- While the kick-start pack exists, it lists C# member spans for a C# plan card.

## PATTERN changes

- `PATTERN-glyph-conversion-chokepoint` — changes: Converting glyphs and paths: only via `glyph`; a plan ref's or handle's alphabet only via `planparser`.

## Notes for the plan writer

- Quarry `v0.4.0`'s alphabet rules are in quarry's `docs/glyph.md` §2, §3 and §5; read them before writing the D3 helper.
