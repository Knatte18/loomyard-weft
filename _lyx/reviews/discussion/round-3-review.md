---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:152
    summary: "The C# field, const and event head cut skips attribute lists and bracket pairs but not generic type arguments or tuple types, so a field typed Dictionary<string, int> is cut at string."
    fix: "In D7's field, const and event sentence, replace the rule 'the first identifier outside attribute lists and bracket pairs that is followed by a comma, =, ; or the end of the signature' with a depth-aware one. The declarator name is the last identifier at nesting depth zero, outside attribute lists and outside every (), [] and <> pair, before the first depth-zero comma, = or ;, or before the end of the signature. Add one acceptance-criteria row under the C# Rename bullet: renaming a field declared as private Dictionary<string, int> map; derives the head private Dictionary<string, int> renamed, and a tuple-typed field derives a head that keeps its tuple type."
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:141
    summary: The record-batch rewrite of a re-sign's old glyph to its minted id states its order against DetectDrift but not against the plan-fingerprint restamp, and it names record-batch only although the post-batch pass is shared with recover-batch.
    fix: "In D7's record-batch bullet, state where the rewrite sits in the shared post-batch pass that both record-batch and recover-batch run. It runs after the done checks, either inside the RewriteRefs call that handle binding makes or in its own call, and in both cases is followed by restampFingerprint before the scope guard and DetectDrift run. Say that a recovered batch gets the same rewrite. Add an acceptance row: after a C# re-sign batch is recorded or recovered, the next begin-batch is not refused with a fingerprint mismatch."
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:127
    summary: "D7's first bullet sets Owner to the glyph's owner chain whole, the second removes the interface qualifier, and neither says how the head decides which trailing components are the qualifier."
    fix: "In D7's first bullet, change 'the owner from that owner chain as Owner' to 'the owner chain with the interface qualifier removed, per the next bullet'. In the second bullet, define the qualifier as the longest trailing run of owner-chain components that the head writes, dotted and followed by a dot, immediately before the declared member name. A head that does not write them leaves the chain whole, as for a member of a nested type. Add a row to the explicit-interface acceptance criteria for a nested type whose member head names a type of the same spelling, which keeps the full owner."
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:94
    summary: D4's bound misstates path-missing (it says directory and earlier card) and glyph-ambiguous (it omits that a declared namespace is needed).
    fix: "In D4's Bound paragraph, reword the namespace self glyph sentence. path-missing refuses the ref when nothing exists on disk at that path, file or directory, and no card, earlier or later, names that exact path as a Create target or Rename destination. Resolve answers glyph-ambiguous when a same-named directory exists and a file declares the namespace, and found when no file declares it."
  - id: F5
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:137
    summary: resign-overload-collision covers a minted id that resolves found or equals a Create or Rename handle, but not two re-sign arrows in one plan minting the same id from different targets.
    fix: "In D7's resign-overload-collision sentence, add a third refusal. A minted id that equals the minted id of another re-sign arrow in the plan is refused with the same exception for an intervening card that removes one of the two, and its detail names the other arrow's card and the way forward of changing one head to a free signature. Add the case to the acceptance row that lists the collision forms."
  - id: F6
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md:193
    summary: The third Open risk says a slash-free self glyph triggers the .cs scan, but quarry scans for any target that parses as a C# glyph.
    fix: "In the third Open risks bullet, replace 'a resolve batch holding a slash-free self glyph now scans the repository's .cs files' with 'a resolve batch holding any target that parses as a C# glyph, such as a slash-free self glyph, now scans the repository's .cs files'."
---

### [MEDIUM] Field head cut breaks on generic and tuple types

**Location:** `_lyx/discussion/decision-record.md:152`

**Issue:** The one-declarator head for a C# field, const or event is cut before the first declarator's name, found as the first identifier outside attribute lists and bracket pairs that is followed by `,`, `=`, `;` or the end of the signature.
Quarry gives every declarator the whole declaration as its signature, confirmed in `internal/engine/csharp.go`: `field_declaration` and `event_field_declaration` mint with `node: decl` and no body, so `SignatureCut` returns the whole declaration text.
Read with only `[]` as the bracket pair, `private Dictionary<string, int> map;` yields `string` as the declarator, because `string` is followed by a comma, and the derived head is garbage that `Name` refuses or mis-mints.
A tuple type `(int, int) pair;` fails the same way, and `Dictionary<K,V>` fields are common in a C# hub.
The multi-declarator case the decision handles correctly is the rarer one.
The var and const synthesis is correctly scoped to Go: its `sym.Kind` switch would otherwise catch a C# const, and D7 states it applies only to Go symbols.

### [MEDIUM] Order of the re-sign rewrite against the restamp and the shared pass

**Location:** `_lyx/discussion/decision-record.md:141`

**Issue:** The record states that the rewrite runs before `DetectDrift`, that it also applies to the in-memory plan, and that `ScopeGuard`'s union holds both ids, so the ordering against drift and scope guarding is stated.
It does not state the plan-fingerprint restamp.
In `internal/websterengine/recordbatch.go` the post-batch pass restamps after each plan rewrite, and `internal/websterengine/fingerprint.go` documents that a rewrite without a restamp makes every later begin-batch fail with `ErrFingerprintMismatch`.
The pass says "No third restamp: the two above already cover every rewrite this pass can perform", so a third rewrite needs its own restamp or must ride inside the binding call.
The pass is also shared by record-batch and recover-batch, and its own comment records that skipping recover-batch left later cards pointing at unbound handles.
The record names record-batch only.

### [MEDIUM] Owner derivation rule is stated twice and its qualifier test is undefined

**Location:** `_lyx/discussion/decision-record.md:127`

**Issue:** One rule is stated for every declaration kind, and it fits quarry: the second bullet strips the interface qualifier from the owner chain, because quarry's `Name` refuses an owner component with type arguments and reads the qualifier from the head, which matches `nameCSharpContainer` and `checkMintedContainer` in `internal/engine/name.go`.
Two defects remain.
The first D7 bullet still says the handle's `Owner` is the owner chain, which reads as no stripping.
The second bullet says the head "spells" the trailing components as a qualifier without a test.
Quarry itself only checks that the head contains the dotted extra components anywhere, so a loose test in LoomYard strips a nested owner that a member head's type merely repeats, and `Name` then mints an id missing that owner.
For a re-sign that surfaces as `resign-head-mismatch`, and for a Create handle as a later bind-count mismatch.

### [LOW] D4's bound misdescribes what path-missing and resolve do

**Location:** `_lyx/discussion/decision-record.md:94`

**Issue:** `checkPathMissing` in `internal/planparser/validate.go` is satisfied by `os.Stat` on the path, so a file or a directory, and by the union of every card's Create targets and Rename destinations.
`createTargetsUnion` and `renameTargetsUnion` span the whole plan, not earlier cards.
The text says "no directory of that name exists and no earlier card creates or renames one into place".
It also says resolve answers `glyph-ambiguous` when a same-named directory exists.
Per quarry `docs/glyph.md` §5 that needs a file declaring the namespace as well; with none declared, the Go path answer stands as `found`.

### [LOW] Re-sign collision check leaves out two re-signs minting one id

**Location:** `_lyx/discussion/decision-record.md:137`

**Issue:** The record covers a minted id that resolves found and one that equals a Create handle's or Rename new handle's canonical id, which settles the focus question.
`CanonicalizeHandles` keeps re-sign answers out of its canonical-owner bookkeeping, so two re-sign arrows on different overloads that mint the same id are caught by neither rule.
Both cards pass their plan check and the second edit would declare a duplicate overload.

### [NIT] Open risk understates what triggers the .cs scan

**Location:** `_lyx/discussion/decision-record.md:193`

**Issue:** Quarry `docs/glyph.md` §5 says each call holding a target that parses as a C# glyph scans the `.cs` files, so a slash-free Go member glyph triggers it too.
The risk names only a slash-free self glyph.
It is a cost note, so only the wording changes.

## Rejected

- The first Open risk, that quarry's C# round trip over a real C# checkout has not been run: not a deferred research question, since the design takes quarry's answers as given and states what LoomYard does with a miss, and the check lives in quarry's own gated round-trip test.
- D3's answer for a not-yet-existing slash-free root path self glyph: correct against `mergeResolve` in quarry's `internal/engine/resolve.go`, which gives `not_found` with `unit: not_found`, and against `createFindings` in `internal/planglyph/create.go`, which passes it with only `create-new-unit`.
- D1's `Lang` sources and D7's no-pair rule for constructors: consistent with quarry's `Declaration` contract and with D3.
