---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/dotnetimpact/lex.go:208
    summary: A verbatim string opening with an escaped quote is lexed as a raw string and swallows following code, silently dropping identifiers and under-narrowing.
    fix: In `stringBody`, take the `quotes >= 3` raw-string branch only when `!verbatim`; for a verbatim string consume the opening quote and run the verbatim loop, where `""` is an escaped quote. Add rows to `TestParseSource` in `internal/dotnetimpact/source_test.go` for `@""""` and `@"""Hidden ""x"" """` followed by a type reference that must still appear in `Identifiers`.
    origin: lens:code-correctness
  - id: F2
    severity: LOW
    class: design
    location: internal/dotnetimpact/source.go:262
    summary: An identifier `record` followed by another identifier (`foreach (var record in rs)`, `record is Foo`) is parsed as a record declaration, adding a bogus type named `in` or `is` that pulls nearly every test class into the closure.
    fix: In `startsTypeDecl`, for the `record` keyword require the name to be followed by `(`, `<`, `:`, `{` or `;`, or reject the names `in`, `is`, `as` and `when`; add a `TestParseSource` row with a `foreach (var record in rs)` body asserting no type named `in`.
    origin: lens:code-correctness
  - id: F3
    severity: LOW
    class: consistency
    location: internal/gatecli/doc.go:181
    summary: The doc says only a whole entry carries under the `none` scoper, but under `dotnet` a narrowed (partial) pass is never recorded either, so `dotnet` belongs beside `none`.
    fix: In internal/gatecli/doc.go:181 change "under the `none` scoper only a whole entry carries" to name `dotnet` too, and say that a dotnet pass narrowed to a project and its classes records nothing.
    origin: lens:code-affected-set
---

### [MEDIUM] Verbatim string starting with an escaped quote is read as a raw string

**Location:** internal/dotnetimpact/lex.go:208
**Issue:** `stringBody` counts the opening run of quotes and sends three or more to `rawBody`, ignoring the `verbatim` flag. In `@"""Foo"" bar"` the run of three quotes is `@"` plus an escaped `""`, yet the lexer looks for a closing `"""`. With none nearby it scans to a later `"""` or the end of the file, so every identifier after that point is missing from `Identifiers` and from the type declarations. The closure then misses test classes, which narrows to too few classes without any fallback, the failure the decision record's Decision 3 rationale says must widen. No existing test covers a verbatim string that opens with `""` (`source_test.go:31` only has an inner `""`).

### [LOW] `record` used as an identifier is parsed as a type declaration

**Location:** internal/dotnetimpact/source.go:262
**Issue:** `startsTypeDecl` accepts the `record` keyword whenever an identifier follows, so `var record in rs` or `record is Foo` declares a type named `in` or `is`. `Identifiers` holds keywords, so nearly every file names `in`/`is`, and a changed file with such a line selects most test classes. The direction is safe (over-selection) but defeats the narrowing.

### [LOW] Gate CLI doc names only `none` as whole-entry-only

**Location:** internal/gatecli/doc.go:181
**Issue:** `recordPass` skips partial invocations, which the dotnet scoper produces, so under `dotnet` only whole entries carry as under `none`; the doc sentence leaves `dotnet` out.

## Rejected

- A `decodeBuild` row for a `build` holding only `about`, `dir` or `env` (code-tests lens): the refusal for a missing `command` is already pinned by the missing-command row, so the extra row duplicates it.
- A possible missing assertion that blocked tiers print no `tier<N>` summary line (code-tests lens): the lens did not open the test and named only a suspicion; the code in `reportBlocked` prints no such line.
- Not read by the lens: `traceFailure` on a build-failure log in `internal/landingshed/verifygate.go:99`; no defect was shown, so none is raised.
