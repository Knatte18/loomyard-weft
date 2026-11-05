```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens, running each changed function or branch on a concrete input.

**Per-function list**

- `signatureMismatches` (`donecheck.go:188`):
  - A C# key `Shop#CartTests.Adds(int)` against a declared `Adds(string)` under the same unit and owner returns that one declaration. Correct.
  - A Go key or a self glyph returns none, because Go `Params` is always nil (quarry `glyph.go:33`) and `IsSelf` is true for a path. Correct.
  - A key `Adds()` has a non-nil empty `Params`. Against a declared `Adds(int)` it counts as a mismatch. Against a declared nil-`Params` member it counts as equal, because `slices.Equal(nil, []string{})` is true. In both cases the result is correct.
  - A key with no parameter list returns none. Correct.
  - Different owner or different unit returns none. Correct.
- `signatureMismatchFinding` (`donecheck.go:204`):
  - The caller guards `len > 0`, so `mismatched[0]` cannot panic.
  - A Create entry gives the "Create target" subject and a `rename-not-done-new` entry gives "Rename pair's new side". Correct.
  - All mismatched symbols share the unit, so using `mismatched[0].Glyph.Unit` for nearness is correct.
- `doneCheckVerdicts` order (`donecheck.go:393-404`):
  - The counterpart-package check runs first and `continue`s on a hit. The mismatch check follows, then `searchedDescription`.
  - The counterpart check only applies to Go, and the mismatch only fires for C# parameter lists, so the two never compete.
  - Create and Rename new side both go through this one branch, so they behave the same.
  - The mismatch finding skips `lookupDescription` and the "still does not resolve" prefix. That matches issue #61, which asks it to lead with the declared signature.
  - The mismatch check does not look for an exact-parameter-list declaration among `declarations`. It relies on quarry's `not_found`, so it cannot fire on a resolvable key.
- `searchedDescription` (`donecheck.go:219`):
  - Zero unit files gives no "searched" part, and the declared-elsewhere part still follows. Correct.
  - One unit file gives the singular wording. Many files give the count, a set-based dedup, with no listing. Correct.
  - Zero declarations elsewhere gives "no declaration named X exists anywhere". Correct.
  - A self glyph or parse error returns `""`. Correct.
- `declarationList` (`donecheck.go:254`):
  - Empty input returns `""`. No caller passes empty input.
  - Exactly three declarations gives no "and N more". Four gives "and 1 more". Correct.
  - The sort key is shared-unit-segment count, then glyph ID, so the order is deterministic. It sorts a clone, so the input is not mutated.
  - The cap of three holds on every path that uses it, including the mismatch path, because the signature list also goes through it.
- `sharedUnitSegments` (`donecheck.go:274`): Empty units give 0, and equal units give the full segment count. Correct. Splitting on `.` also splits a Go unit such as `cmd/foo.bar` into extra segments. This is harmless, since it only affects ordering.
- `cardNotDoneFor` (`recordbatch.go:891`): The two new flags are set per finding code and do not disturb the existing flags. Correct.
- `failCardNotDone` precedence (`recordbatch.go:208-224`):
  - A mix with delete-not-done where a later card references the target takes the delete way forward. Correct.
  - A mix with glyph-rejected takes the glyph way forward, which also amends the card. Correct.
  - A mix of create-not-done and signature-mismatch takes the amend-only route. Correct.
  - Create-not-done alone on `record-batch` gives both routes. Alone on `recover-batch` it gives amend only. Correct.
  - The uncheckable branch gives the fresh-restart steps and leads the function, consistent with the other way forwards.
- `createTargetWayForward` (`recordbatch.go:135`): The amend text names `recover-batch NN` as its last step, not as the whole way forward. Issue #62 is met, since the way forward no longer points only at the verb that just refused.

**Findings**

1. LOW, `design`, `internal/websterengine/recordbatch.go:139`. A signature-mismatch from a Rename new side (`donecheck.go:206`) reaches `createTargetWayForward`, whose text says "amend the card's Create target". For a Rename card the target is the pair's new side, so the instruction misdirects the reader. Evidence: `donecheck.go:204-215` can emit the finding for `rename-not-done-new`, and `cardNotDoneFor` (`recordbatch.go:907`) flags it without telling the two cases apart.

2. NIT, `design`, `internal/websterengine/recordbatch.go:223`. `recovered` is derived from `in.Verb == "recover-batch"`. A first recovery whose strand is the first to hit create-not-done is therefore treated as "a recovery already tried", and the `recover-batch` hint is dropped. Only the amend route is offered, which still makes progress, so this is not a defect. It only means the recover-batch hint is lost on a first-time create-not-done found during recovery.

I found no other wrong result, swallowed error or inconsistent state on a failure path in the changed code.

**Tool calls**
1. Read `internal/planparser/glyphref.go`: needed `ParseRef`'s behaviour for `Params` and self glyphs, which the manifest lacked.
2. Grep of the quarry `glyph` package for `Params`: needed the nil versus empty `Params` semantics and the fact that Go `Params` is always nil.
