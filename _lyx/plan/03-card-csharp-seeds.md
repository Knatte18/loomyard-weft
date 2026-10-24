# Card 3 — card seeds for the dotnet scoper from the plan parser

**Create:**
- `internal/planparser#CSharpSeed`
- `internal/planparser#CardCSharpSeeds`
- `internal/planparser_test#TestCardCSharpSeeds`

**Extends:**
- `internal/planparser/doc.go`

**Uses:**
- `internal/planparser#RefFile`
- `internal/planparser#ParseRef`
- `internal/planparser#HandleBody`
- `internal/planparser#Plan`
- `internal/planparser#Card`
- `internal/planparser#TargetGroup`

**Intent:**
Give the `dotnet` scoper's `--card` form its seeds from the sole plan parser (Decision 2's card seed, Constraint `PATTERN-sole-parsers`), since `CardPackages` is Go-shaped and maps only self glyphs to paths.

`CSharpSeed` is one card Target: `Ref`, the target as the card spells it; `File`, the worktree-relative slash path it names, empty when it names none; and `Types`, the type names it seeds.
`CardCSharpSeeds` returns one seed per target ref of every target group of `cards`, in card and body order, every group type included, a `Rename` group contributing both sides of each pair, and `Uses` left out, since it is read rather than written.

- A ref `RefFile` maps to a file, a path with an extension or a file self glyph, seeds that `File`.
- A member glyph or `plan:` handle, parsed through `ParseRef` after `HandleBody` strips a handle, seeds each name of its owner chain, outermost first (`NORCE.Models#Foo.Bar` seeds `Foo`, `NORCE.Models#Outer.Inner.M` seeds `Outer` and `Inner`).
- A glyph with an empty owner chain seeds its own name as a type, since C# has no namespace-level members (`NORCE.Models#Circle` seeds `Circle`).
- Each seeded name has its generic or arity suffix stripped at the first `<` (`Box<T>` and `Box<,>` seed `Box`), and a leading `@` dropped.
- Any other ref, such as a directory path or a namespace-only self glyph, yields a seed with neither `File` nor `Types`, which the scoper reads as a fallback to whole.

The function reads no disk: existence, inert patterns and the `.cs` extension are the scoper's to judge.
`doc.go` names the function beside `CardPackages`.

Tests:
`TestCardCSharpSeeds` is a table over in-memory `language: csharp` plans built from `Plan` and `Card` values with `TargetGroup` refs: a `.cs` path, a file self glyph, a member glyph, a nested member glyph, a type glyph, a generic type glyph, a `plan:` handle, a `Rename` pair, a `Uses` entry left out, a directory path and a namespace self glyph yielding an empty seed, each row asserting the seeds exactly.

**ImpactSummary:** one new exported function and type in `planparser`; no existing member changes, so no caller is affected.
