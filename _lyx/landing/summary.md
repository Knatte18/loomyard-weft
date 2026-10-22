# Plan glyphs for C#: quarry v0.4.0 and `language: csharp`

LoomYard's planner, Plan-Review and the plan gates now work with C# glyphs in a C# hub such as NORCE Models.
Every Go plan behaves as before: `language:` still defaults to `go`.

## What changes

- `go.mod` pins `github.com/Knatte18/quarry` at `v0.4.0` (from `v0.2.1`, so it also takes in `v0.3.0`'s Enclose, which nothing uses yet).
  quarry now rejects a declaration with a zero `Lang`, so every declaration LoomYard builds sets it: handles, re-sign arrows and Rename.
- A plan's `language:` key accepts `csharp` beside `go` and `none`.
  `ReviewExempt` maps `csharp` to `.cs`.
- `internal/planparser` gains one helper that picks the alphabet for each plan ref and `plan:` handle body.
  `internal/planparser`, `internal/planglyph` and `tools/plandag` parse refs only through it.
  Under `csharp` a ref both alphabets parse is read as C#, a Go-only ref is accepted only as a path self glyph such as `src/Foo.cs#`, and any other ref is `glyph-malformed`.
  Path entries still canonicalize through the Go unit alphabet, since quarry spells paths only that way.
- New findings, each naming its way forward:
  - `plan-language-mismatch` (blocking): the plan's source files are the other language's, or a `go` plan's member glyphs resolve only into `.cs` files.
    A `glyph-malformed` ref the C# alphabet parses under `go` also names `language: csharp`.
  - `rename-namespace-pair` (blocking): a pending Rename pair names a C# namespace, such as `Norce.Models# -> Norce.Core#`.
  - `resign-overload-collision` (blocking): a C# re-sign mints an id that an existing overload, a Create or Rename handle, or another re-sign already has.
  - `resign-not-done`: after a C# re-sign batch is recorded or recovered, the minted id does not resolve `found`.
  - Cross-alphabet `glyph-ambiguous` for a self glyph, naming both languages.
- C# handles, re-sign arrows and Renames:
  - A handle binds with `Unit`, `Owner` and `Decl` taken from the C# parse, not from a split at the last `.`.
    `Owner` leaves out an explicit-interface qualifier.
  - A re-sign is accepted when the minted id equals the target apart from its parameter list.
    A minted id that drops or adds the parameter list is `resign-head-mismatch`.
  - Once a re-sign is recorded, the old glyph is rewritten plan-wide to the minted id and the fingerprint is restamped, so the next begin-batch meets no mismatch.
    The batcher starts a new batch at the first later card naming the re-signed member.
  - A Rename derives its head by substituting the declared name token, and a field, const or event derives a one-declarator head.
    A constructor Rename is refused (`rename-old-unresolved`), since constructors follow the type.
  - A symbol `Rename` needs a C# entry in the hub's `language.yaml` `rename_tools`.
- Under `csharp`, caller coverage, the delete-order import scan, the verify-package checks, the `.go` handle rule and the external-test unit mapping are skipped.
  `classifyAmbiguity` partitions only all-`.go` candidate sets, so `Foo_windows.cs` is never read as a build constraint.
- The planner stencil, Plan-Review rubric, `contracts/specs/loom-plan-spec.md`, the docs of `internal/planparser`, `internal/planglyph` and `internal/websterengine`, and `pattern/PATTERN-glyph-conversion-chokepoint.md` describe the C# alphabet.
  The chokepoint entry in `PATTERN.md` says a ref's alphabet is chosen only in `planparser`; two other overview entries (`PATTERN-testkit`, `PATTERN-review-round`) were shortened to keep the overview within its byte limit, with their text already in the background files.

## Design decisions

- The plan names its alphabet once, by file extension, rather than per ref or per hub.
  A plan that changes Go and C# code at once sets `language: none`.
- Under `csharp`, a slash-free self glyph such as `README.md#` or `docs#` parses as a namespace and has no `UnitPath`.
  Its existence is checked only by quarry's resolve, the batcher costs no file for it, and `RefCovers` maps no C# member or namespace glyph to a scope.
  `internal/planparser/doc.go` lists the full bound.
- No Go-toolchain check gets a C# counterpart; .NET verify gates and Enclose wiring stay with their own board entries.

## Check by hand

- Quarry's C# round trip has not run over a real C# checkout, so a C# answer may miss a symbol; LoomYard reports it as `not_found`.
- A directory and a namespace sharing a spelling make self glyphs resolve `ambiguous`; the planner must name a file or a member.
- The `.cs` fixtures show up in `lyx quarry` listings of their directory in this Go hub, and a resolve batch holding a C#-parsing target scans the repository's `.cs` files.
- Review findings left open, to check in the code:
  - Explicit-interface member owner: `Owner` for `void System.IDisposable.Dispose()` and `int IProvider<A>.Value` in `internal/planglyph/handle.go` and `internal/planglyph/csharphead.go`.
  - Field head cut with generics and tuples: `internal/planglyph/csharphead.go`.
  - Re-sign batch order against drift check and scope guard: `internal/websterengine/recordbatch.go`, `internal/batcher/boundary.go`.
  - A `go` plan whose handles are all `plan:` handles and whose resolve-half mismatch check applies: `internal/planglyph/languagemismatch.go`.
  - Glyph parsing outside `planparser`: `internal/planglyph/resign.go` parses minted ids through `planparser.ParseMintedCSharpID`; confirm no other `glyph.Parse` call remains outside `planparser`.
  - `PATTERN.md` edits to `PATTERN-testkit` and `PATTERN-review-round` that the glyph change did not declare.
- The plan-level verify first failed on `internal/loomcli.TestGateParity_PlanGate/VerifyNestedModule`; the fix commit `55eb851f9` runs the parity fixture's nested-module verify row under `language: go`.
  Check that the fixture change is the right repair and not a masked regression.
