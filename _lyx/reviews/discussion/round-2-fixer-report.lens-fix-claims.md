```yaml
lens: fix-claims
outcome: reported
```

The fix-claims lens checked every claim the fix added or rewrote against the code at HEAD. I found one LOW finding and nothing else.

**[LOW] A type glyph's name can carry a generic suffix**
- **Changed text:** `decision-record.md` Decision 2, Card seed bullet: "for a glyph whose owner chain is empty, its own name as a type … (`NORCE.Models#Circle` seeds `Circle`)".
- **Summary:** The record says to seed the glyph's name as a type, but that name can end in `<...>`, which no identifier token the lexer yields will match.
- **Evidence:**
  - quarry v0.4.0 `glyph/csharp.go` `checkCSharpName` accepts "an identifier with an optional arity suffix".
  - `checkCSharpGenericIdentifier` accepts an identifier "optionally followed by `<...>`".
  - So `NORCE.Models#Box<T>` parses with `Name` `Box<T>`, and the seed would be `Box<T>` where the sources spell `Box`.
  - The owner-chain seed already had the same gap, since `checkCSharpOwner` accepts the same suffix on each component, but the fix repeats it for the new case.
  - The sentence should say the seed is the name, and each owner component, with its `<...>` suffix stripped.

**Claims that hold:**
- `NORCE.Models#Circle` parses with `Owner` nil and `Name` `Circle`, per `checkCSharpMember` with a single component.
- `gateslot.ConfigOpenMaps` returns `tiers` and `tier_parallel` only.
- `yamlengine.Merge` carries the keys at an open-map path whole, so adding `build` to `ConfigOpenMaps` carries `command`, `dir` and the other sub-keys as whole units.
- `impactset` reads untracked files with `git ls-files --others --exclude-standard -z` (`internal/impactset/targets.go:102`).
- `runTest` keeps the first failure's code (`internal/gatecli/run.go:137-148`).
- `scope.forTier` exists (`internal/gatecli/scope.go:72`).

**Tool calls:**
1. Read quarry `csharp.go` field lines and `internal/configengine` open-map uses, to check the C# glyph `Owner`/`Name` claim and the open-map carry claim.
2. Read `internal/gateslot/config.go` categories, to check that `build` would sit beside `tiers` as an executable key.
3. Read the `yamlengine.Merge` doc comment, to check that "carries its sub-keys whole" matches open-map merge semantics.
4. Read quarry `checkCSharpMember`, to check how a type glyph splits into owner and name.
5. Read quarry `checkCSharpOwner`, `checkCSharpName` and `checkCSharpGenericIdentifier`, to check what suffix a name component can carry.
