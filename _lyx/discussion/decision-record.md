# Decision record — quarry-csharp

## Goal

LoomYard takes quarry `v0.4.0` and plans, reviews and implements with C# glyphs in a C# hub such as NORCE Models, with every Go plan behaving as today.

## Scope

In:

- Raise `github.com/Knatte18/quarry` in `go.mod` from `v0.2.1` to `v0.4.0`, taking in `v0.3.0`'s Enclose unused, and fix the library break: every `quarry.Declaration` sets `Lang`.
- A plan's `language:` key gains the value `csharp`, beside `go` (still the default) and `none`.
- The rule that picks the alphabet for each plan ref and handle, the extension guard on the plan's language, and the C# side of `plan:` handles, re-sign arrows and Renames.
- Reading quarry `v0.4.0`'s multi-alphabet answers: cross-alphabet ambiguity, a C# self glyph's `listings`, the new Kind values.
- The planner stencil, Plan-Review rubric and the plan spec describe the C# alphabet; the package docs of `internal/planparser` and `internal/planglyph` describe the new rules.

The board entry asks for the alphabet "per file, by extension".
A C# member glyph names a namespace and carries no file, so the alphabet is named once per plan and picked by extension at two points: the planner picks `language:` from the extension of the files it plans (D2), and D4 checks that pick against the plan's file targets.
Path entries take the Go unit alphabet whatever their extension, because quarry spells a path only in that alphabet (D3).

Out:

- Wiring Enclose into any check: board entry `enclose-checks` owns that, and this raise is the `go.mod` bump that entry names.
- C# counterparts of the Go-toolchain checks (caller coverage, delete-order import scan, build-constraint partitioning, verify-package checks): no board entry asks for them.
- A plan that uses glyphs for Go and C# code at once; such a plan sets `language: none`.
- Renaming a C# namespace through a `Rename` pair.
- .NET verify gates (`dotnet-scoped-verify`, `dotnet-verify-protocol`), language lens sets (`lenses-language-sets`), the kick-start pack's removal (`kickstart-pack-remove`).

## Decisions

### D1 — Raise quarry to v0.4.0 and set `Lang` on every declaration

Decision: `go.mod` pins quarry `v0.4.0`.
Every `quarry.Declaration` LoomYard builds sets `Lang`: the plan's language for a Create handle, and the resolved glyph's language for a re-sign arrow or a Rename's derived declaration.
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
`Plan.GlyphLanguage` keeps only its glyph-enabled answer; no caller parses with a language value of its own.
Under `go` the helper parses with `glyph.Go`, exactly as today.
Under `csharp`:

- a ref the C# alphabet parses is a C# glyph, including a ref both alphabets parse;
- a ref only the Go alphabet parses is accepted only when it is a self glyph: a path self glyph such as `src/Foo.cs#`;
- any other ref is `glyph-malformed`, with the C# alphabet's parse error.

Path-shaped entries canonicalize through `glyph.Self(glyph.Go, path)` under both `go` and `csharp`, because quarry spells a path only in the Go unit alphabet.

Bound: under `csharp`, a slash-free self glyph such as `README.md#`, `Program.cs#` or `docs#` parses as a C# namespace, and so has no `UnitPath`.
What that lets through, all of it:

- `card-path-malformed` and `path-missing` skip it; its existence is checked only by the resolve pass, where quarry tries both alphabets.
- `ReviewExempt` counts it as a source target, so such a plan goes to Plan-Review.
- The batcher's card estimate and the target-directory derivation count no file for it, nor for any C# member or namespace glyph; a C# card is costed by its slashed path targets only.
- The redundancy check sees it as a namespace, so it raises no package-versus-member redundancy for it.

A rename pair whose two sides are both self glyphs is a file pair under `csharp`, so `Program.cs -> App.cs` still runs `git mv`, as under `go`.
A slashed path self glyph keeps every path check.

Rationale: quarry's own resolve tries every alphabet, but LoomYard's checks need one parsed glyph per ref; reading the overlap as C# costs only the bounded items above, and one helper keeps the choice in the package that owns ref shapes.

### D4 — The plan's language is checked against its files' extensions

Decision: a new blocking plan finding, `plan-language-mismatch`, fires when a plan's path targets name source files of the other glyph language (`.cs` under `go`, `.go` under `csharp`) and none of its own language.
It reads the canonicalized path entries in their Go-alphabet reading, never D3's, so a root `tool.go` counts under `csharp`, and it reads no disk.
Its detail names the way forward: set `language:` to the language of the files, or `language: none` for a plan that changes both.
It does not fire under `none`.

Under `go`, a `glyph-malformed` ref that the C# alphabet parses gets a detail naming `language: csharp` as the way forward.

Bound: a Go plan that edits `.cs` fixtures beside any `.go` file is not refused; a plan whose only source targets are the other language's files is.

Rationale: a C# hub's planner that leaves the default would otherwise meet `glyph-malformed` findings that do not say what is wrong, whether its cards name `.cs` paths or only member glyphs.

### D5 — Go-toolchain checks run only under `language: go`

Decision: the checks that read Go source or Go toolchain semantics run only when the plan's language is `go`: caller coverage (`go/types`), the delete-order import scan, build-constraint partitioning of ambiguous answers, the verify-package checks, the `.go` file-unit handle rule and the external-test unit mapping (`goPackageDir`).
Under `csharp` they are skipped.

Bound: a C# plan gets no caller-coverage, import-order or verify-package finding; every glyph check that rests on quarry's answers (resolve, Create, done, scope, drift, delta) still runs.

Rationale: these checks key on `.go` files and the Go toolchain and have no meaning for a `.cs` file.

### D6 — LoomYard reads quarry v0.4.0's multi-alphabet answers

Decision:

- A cross-alphabet `ambiguous` answer for a self glyph, with no candidates and a `languages` list, yields `glyph-ambiguous` whose detail names the languages and quarry's way out: name the file or a member.
- A member glyph's cross-alphabet `ambiguous` answer is handled like any other ambiguous answer; build-constraint partitioning (D5) applies only to Go candidates.
- Wherever LoomYard reads a self glyph's directory listing, it reads `listing` or `listings`, whichever the answer carries, never choosing by D3's parse: `README.md#` read as C# by D3 still answers with the Go path's `listing`.
- Every switch over quarry's Kind accepts every value `v0.4.0` adds.

Rationale: these are the answer shapes `v0.4.0` adds; a reader written for `v0.2.1` would treat an empty candidate list or a missing `listing` as an error.

### D7 — `plan:` handles, re-sign arrows and Renames under `csharp`

Decision:

- A C# handle is `plan:<namespace>#<owner.member>` with a C# declaration head.
  Its namespace, owner chain and name come from D3's helper parsing the handle's body with the C# alphabet, never from splitting the member at its last `.`.
  Binding hands quarry's `Name` a declaration with `Lang: csharp`, the namespace as `Unit`, the owner chain as `Owner` and the head as `Decl`.
- A C# re-sign arrow is accepted when the id `Name` mints equals the target glyph apart from its parameter list, since C# parameter types are part of the glyph.
  The minted id is the target's glyph after the edit, and the done check and drift expect it.
- A C# Rename's derived declaration takes the owner from the resolved glyph and the head from the resolved symbol's signature, matching the glyph's name without its generic arity suffix (`Box<>` against `class Box<T>`).
  A type's constructors follow the type through the hub's rename tool, with no pair of their own.
- A C# symbol `Rename` needs a C# entry in the hub's `language.yaml` `rename_tools`; `rename-tool-missing` already names the way forward.

The Go handle, re-sign and Rename rules stay as they are under `go`.

Rationale: quarry `name` needs the owner for a C# member, since a C# head states no owner as a Go method states its receiver, and a C# member's spelling holds parameter lists and dotted types that a string split cuts in the wrong place.

### D8 — Planner, Plan-Review and the kick-start inherit C# glyphs; no new mechanism

Decision: the `lyx quarry` verbs pass C# answers through unchanged, since quarry walks `.cs` files itself.
The kick-start pack and every plan gate take their alphabet from D3, so they get C# glyphs once the plan says `csharp`.
The planner stencil and `contracts/specs/loom-plan-spec.md` describe `language: csharp`, that a C# member glyph is copied from a quarry answer with its parameter list, that a `.cs` file is written as a plain path, and D4's finding.
The Plan-Review rubric's rule that an `Edit` names Go file paths is worded for both languages.

Rationale: the board entry's three consumers already read glyphs only through quarry and the plan's alphabet, so the work is the alphabet choice and the words that teach it.

## Constraints

- `PATTERN-glyph-conversion-chokepoint`: `glyph.Self` stays the only path-to-glyph call and `Glyph.UnitPath` the only glyph-to-path call; D3 adds no local grammar, and a C# glyph is never read as a path.
- `PATTERN-ref-shape-registry` and `PATTERN-sole-parsers`: the alphabet choice lives in `internal/planparser`, the plan's sole parser.
- `PATTERN-refusal-way-forward`: `plan-language-mismatch`, the `glyph-malformed` detail of D4 and the cross-alphabet `glyph-ambiguous` detail name the way forward, and a test reaches each.
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
- Under `csharp`, a slash-free root path self glyph that does not exist yet reaches resolve as both a missing path and a missing namespace; a Create card's expectations must hold for that answer.
- A repository where a directory and a namespace share a spelling makes self glyphs resolve `ambiguous`; the planner must then name a file or a member.
- The `.cs` fixtures this task adds show up in `lyx quarry` listings of their directories in this Go hub, and a resolve batch holding a slash-free self glyph now scans the repository's `.cs` files.
- C# batch estimates undercount cards that name only member glyphs (D3's bound).

## Acceptance criteria

- `go.mod` pins quarry `v0.4.0`, and every Go plan fixture yields the same findings as before.
- Binding a Go handle, a Go re-sign arrow and a Go Rename succeeds with `Lang` set.
- Under `csharp`, a Create handle whose member has a parameter list holding a dotted type binds to a C# glyph with the right owner and name; a C# re-sign that changes a parameter type is accepted and its minted id is what the done check expects; a C# Rename of a generic type derives its declaration.
- Under `csharp`: a C# member glyph with a parameter list, a C# namespace self glyph and a path self glyph of a `.cs` file each parse and resolve; a ref neither alphabet takes is `glyph-malformed`.
- Under `csharp`, `README.md` as a path target canonicalizes to `README.md#`, skips the path checks, resolves `found` through quarry, and its listing is read.
- Under `csharp`, the root file pair `Program.cs -> App.cs` is a file pair.
- Under `go`, a C# member glyph is `glyph-malformed` with a detail naming `language: csharp`.
- `plan-language-mismatch` fires for a plan whose only source targets are `.cs` under `go`, or `.go` (a root `tool.go` included) under `csharp`; it does not fire for a Go plan with `.go` and `.cs` targets, nor under `none`; it names the way forward.
- Under `csharp`, caller coverage, the delete-order import scan and the verify-package checks report nothing.
- A cross-alphabet ambiguous self glyph yields `glyph-ambiguous` naming both languages.
- `ReviewExempt` treats a `.cs` Prosa target as source under `csharp`.
- While the kick-start pack exists, it lists C# member spans for a C# plan card.

## PATTERN changes

- `PATTERN-glyph-conversion-chokepoint` — changes: Converting glyphs and paths: only via `glyph`; a plan ref's or handle's alphabet only via `planparser`.

## Notes for the plan writer

- Quarry `v0.4.0`'s alphabet rules are in quarry's `docs/glyph.md` §2, §3 and §5; read them before writing the D3 helper.
- The pattern background file `pattern/PATTERN-glyph-conversion-chokepoint.md` gains the alphabet rule and the fact that path self glyphs use the Go unit alphabet under every glyph-enabled language.
