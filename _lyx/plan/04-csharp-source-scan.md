# Card 4 — internal/dotnetimpact: a C# lexer and the source model of one file

**Create:**
- `internal/dotnetimpact/doc.go`
- `plan:internal/dotnetimpact#token` -> `type token struct`
- `plan:internal/dotnetimpact#lex` -> `func lex(data []byte) []token`
- `plan:internal/dotnetimpact#typeDecl` -> `type typeDecl struct`
- `plan:internal/dotnetimpact#sourceFile` -> `type sourceFile struct`
- `plan:internal/dotnetimpact#parseSource` -> `func parseSource(rel string, data []byte) sourceFile`
- `plan:internal/dotnetimpact#TestMain` -> `func TestMain(m *testing.M)`
- `plan:internal/dotnetimpact#TestParseSource` -> `func TestParseSource(t *testing.T)`

**Intent:**
Start the `dotnet` scoper's package, `internal/dotnetimpact`, the sibling of `internal/impactset`, with the pure read of one C# source file (Decisions 1 and 2).
The package is told absolute paths and never imports `internal/lyxcwd` (`PATTERN-cwd-resolution`).

`lex` turns C# source into identifier and punctuation tokens with their positions.
It drops `//` and `/* */` comments, character literals and the literal text of every string: regular, verbatim `@"…"` with its doubled quotes, and raw `"""…"""` with any quote count.
Inside an interpolated string, regular `$"…"`, verbatim `$@"…"` or `@$"…"` and raw `$"""…"""` with any count of `$`, it keeps the code of every hole as tokens, nested strings and braces included, and drops the literal text and the `{{`/`}}` escapes, so `$"{Formatter.Format(v)}"` yields `Formatter`, `Format` and `v`.
A `#` preprocessor line is dropped whole.

`parseSource` walks the tokens and returns a `sourceFile`: `Rel`, the file's slash path relative to the tier's `dir`; `Identifiers`, the set of every identifier token; and `Types`, one `typeDecl` per declared `class`, `struct`, `interface`, `record`, `record struct` or `enum`.
A `typeDecl` holds `Name` (without its type parameters), `Namespace` (from a block or file-scoped `namespace`, nested blocks joined by `.`), `Outer` (the enclosing type names, outermost first), `Kind`, `Abstract`, `Base` (the first name of its base list, generic arguments and namespace qualifier stripped, empty when none), `TestAttributed` (the type carries `TestFixture` or `TestClass`) and `HasTestMethod` (a member carries `Test`, `TestCase`, `TestCaseSource`, `Theory`, `Fact`, `TestMethod` or `DataTestMethod`).
Each attribute is matched in its plain, `Attribute`-suffixed and namespace-qualified spelling (`NUnit.Framework.TestAttribute`), with arguments, and anywhere inside a multi-attribute list (`[Category("x"), Test]`), and an attribute with a target prefix such as `[assembly: …]` sets nothing.
A file-scoped `namespace X;` applies to the rest of the file.

`doc.go` opens the package doc: what the package derives, that it resolves no cwd, and the lexer's rules above; later cards add the selection, the fallbacks and the entry points.
`TestMain` installs `gitkit.HermeticGitEnv` and runs under `tmuxkit.Main`, as `impactset`'s does (`PATTERN-test-isolation`).

Tests:
`TestParseSource` is a `tier0` table over source strings, asserting the identifier set membership and the type declarations exactly: a comment naming a type yields no identifier; a regular, verbatim and raw string naming one yields none; each interpolated form keeps its holes' identifiers, a raw `$$"""` form with `{{x}}` holes included; a block and a file-scoped namespace; a nested class with its outer chain; a generic `class Box<T>` named `Box`; an abstract class with a generic, qualified base; a record and a struct; each attribute spelling on a class and on a method, inside an attribute list and with arguments; and an `[assembly: …]` attribute setting nothing.
