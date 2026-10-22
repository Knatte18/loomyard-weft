# Fixer report — discussion round 2

## Changes

All changes are in `_lyx/discussion/decision-record.md`.

- **F1** (fixed as proposed): D7 gains one `Owner` rule for every C# declaration (handle, re-sign arrow, Rename): the glyph's owner chain with the trailing components the head spells as the interface qualifier removed, with the `System.IDisposable.Dispose()` and `IProvider<A>.Value` examples.
  The handle-only sentence is gone.
  The re-sign bullet states that the arrow's `Unit` and owner chain come from D3's parse of its target, and the Rename bullet takes its `Unit` and owner chain from the resolved glyph.
  New acceptance criterion: a re-sign arrow on `Ns#Renderer.IProvider<A>.Value` mints its own target id.
- **F2** (fixed as proposed, with the cut point made concrete): a C# field, const or event derives a one-declarator head.
  The head is the resolved signature cut before its first declarator's name, followed by the new identifier, with ` = default` appended for a const.
  The rule names the cut point, covers the event with accessors (quarry cuts its signature before them), and gives `public const int A = 1, B = 2;` → `public const int C = default` as its example.
  The Rename acceptance criterion gains the multi-declarator const case.
- **F3** (fixed as proposed): the open risk on a slash-free root self glyph is deleted.
  D3 now states the `not_found` / `unit: not_found` answer per quarry `docs/glyph.md` §5, and that `createFindings` passes it with at most `create-new-unit`.
- **F4** (fixed as proposed): the D4 bound no longer says such a plan is let through.
  It now says the plan is refused by `path-missing` when no directory of that name exists, or by `glyph-ambiguous` when one does.
- **F5** (fixed as proposed, then corrected by the self-check): the record-batch rewrite runs before `DetectDrift`.
  It also applies to the in-memory plan record-batch hands `DetectDrift`, and `ScopeGuard`'s union for the re-sign card holds both the old glyph and the minted id.
- **F6** (fixed as proposed, then extended by the self-check): `resign-overload-collision` also refuses a minted id equal to a Create handle's or Rename new handle's canonical id.
  The exception is when a card between the two declarations removes the earlier one's member.
  The detail names the declaring card and the way forward.
  The overload-collision criterion gains the case.

## Self-check

- fix-consistency, LOW: the criterion covered only the Create handle half of the F6 extension. Fixed as proposed: the criterion now names a Rename new handle too.
- fix-consistency (outside its lens), explicit-interface event field head may not parse. Disputed, merged with the fix-claims LOW below.
- fix-way-forward, LOW: the D4 bound's `path-missing` and `glyph-ambiguous` refusals don't name `language: csharp`. Fixed with own change: the bound now states that neither detail names it. The fork's alternative, a `path-missing` detail naming `language: csharp`, would add a new finding variant outside this round's scope.
- fix-claims, MEDIUM: `DetectDrift` reads the in-memory `in.Plan` (`internal/websterengine/recordbatch.go:928`), which the on-disk rewrite does not update. Fixed as proposed: the rewrite also applies to that in-memory plan.
- fix-claims, LOW: the D4 `path-missing` ref list left out a Rename old side and the create/rename-into satisfaction. Fixed as proposed: "on any ref `path-missing` checks", plus the earlier-card clause.
- fix-claims, LOW (uncertain): an event with accessors and an explicit interface qualifier gives the head `event EventHandler IFoo.New`, which may not parse. Disputed below.
- fix-proposal, LOW: after the plan-wide rewrite, `ScopeGuard`'s union would lose the old glyph. Fixed as proposed: the union holds both.
- fix-proposal, LOW: the handle half of the F6 check had no between-cards removal exception. Fixed as proposed.

## Disputed

- Self-check fix-claims LOW, explicit-interface event head. Ground: premise false.
  Evidence: the fix-way-forward fork read tree-sitter-c-sharp's `grammar.js`, whose `event_declaration` rule accepts `;` after an `explicit_interface_specifier`.
  So `event EventHandler IFoo.Renamed;` parses under quarry's `";"` completion (`nameCSharp`, quarry `internal/engine/name.go`), and `checkMintedContainer` accepts the qualifier the head spells.

## Deferred

None.
