# Round 3 fixer report — discussion

Every finding's premise held against the code: `internal/websterengine/recordbatch.go` `postBatchChecks`, `internal/planparser/validate.go` `checkPathMissing`, `internal/planglyph/handle.go` and quarry's `checkMintedContainer` and `csharp.go` field minting.

## Changes

All changes are in `_lyx/discussion/decision-record.md`.

- F1, fixed as proposed: D7's field, const and event head cut now takes the declarator name as the last identifier at nesting depth zero, outside attribute lists and every `()`, `[]` and `<>` pair, before the first depth-zero `,`, `=` or `;` or the end of the signature, with a `Dictionary<string, int>` and a tuple example.
  An acceptance row covers both.
- F2, fixed as proposed: D7's re-sign rewrite bullet now names the post-batch pass that record-batch and recover-batch share, and says a recovered batch gets the same check and rewrite.
  It places the rewrite after the done checks, inside binding's `RewriteRefs` call or in its own call, followed in both cases by `restampFingerprint` before the scope guard and `DetectDrift`.
  An acceptance row covers the next begin-batch after a recorded or recovered re-sign batch.
- F3, fixed as proposed: D7's first bullet now passes the owner chain with the interface qualifier removed.
  The second bullet defines the qualifier as the longest trailing run of owner-chain components the head writes, dotted and followed by a dot, immediately before the declared member name, and says a head that does not write them leaves the chain whole.
  An acceptance row covers a nested type `Outer.Item` whose head `Item.Kind Get()` keeps the full owner.
  The self-check refined this rule further (see below).
- F4, fixed as proposed: D4's bound now says `path-missing` refuses a ref when nothing exists on disk at that path, file or directory, and no card, earlier or later, names that path as a Create target or Rename destination.
  It also says resolve answers `glyph-ambiguous` only when a file also declares the namespace, and `found` when none does.
- F5, fixed as proposed: `resign-overload-collision` gains a third refusal for two re-sign arrows that mint the same id, with the intervening-removal exception and a detail naming the other arrow's card.
  The collision acceptance row lists it.
  The self-check narrowed it to arrows on different targets (see below).
- F6, fixed as proposed: the third open risk now says any target that parses as a C# glyph triggers the `.cs` scan.

## Self-check

- fix-consistency: no findings.
- fix-claims: no findings.
- fix-way-forward: no findings.
- fix-proposal, MEDIUM, F5's refusal also caught two re-signs of the same member: fixed as proposed. The refusal now applies only when the other arrow's target is a different glyph, and the acceptance row says two re-signs of the same target are not refused.
- fix-proposal, MEDIUM, F3's qualifier test matched literal text against quarry's canonical spelling: fixed as proposed. The comparison is now in quarry's canonical C# spelling, with an `IProvider<string, int>` example.
- fix-proposal, LOW, F3's rule did not say how the declared member name is found: fixed as proposed. It now uses the Rename bullet's declared-token rule, with a `public Item.Value Value { get; }` example.

## Disputed

None.

## Deferred

None.
