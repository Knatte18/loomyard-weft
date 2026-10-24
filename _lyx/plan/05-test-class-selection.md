# Card 5 — internal/dotnetimpact: the type-name closure, test classes, projects and filter terms

**Create:**
- `internal/dotnetimpact#Scope`
- `internal/dotnetimpact#Project`
- `internal/dotnetimpact#seed`
- `internal/dotnetimpact#narrow`
- `internal/dotnetimpact#filterTerm`
- `internal/dotnetimpact#TestNarrow`

**Extends:**
- `internal/dotnetimpact/doc.go`

**Uses:**
- `internal/dotnetimpact#sourceFile`
- `internal/dotnetimpact#typeDecl`
- `internal/dotnetimpact#parseSource`

**Intent:**
Add the pure core of the derivation: from the scanned sources, the project files and a seed, the test projects and filter terms a narrowed run takes (Decision 2, and the project fallbacks of Decision 3).

`Scope` is the derivation's answer: `Whole` with its `Fallback` reason; else `Projects`, one `Project` per test project holding a selected class, in path order, each with `Path`, the `.csproj` path relative to the tier's `dir`, and `Terms`, sorted and without repeats; and `NoTestClass`, set when a non-empty seed selected no test class.
A `Scope` with no project and `NoTestClass` unset is the empty scope of an empty or inert-only seed, so the caller tells the two empty scopes apart (Decisions 3 and 6).
`seed` holds `Files`, the set of changed or targeted `.cs` paths relative to the tier's `dir`, and `Types`, the seeded type names.

`narrow`:

- Every seed file must have a `.csproj` in some ancestor directory up to the `dir` (`projects` lists every `.csproj` path the scan found, relative to the `dir`), else `Whole` naming the file and that no project holds it.
- The closure starts from the seed's types and the types declared in its files, and adds, until nothing changes, every type declared in a file whose `Identifiers` name a type already in the set.
- A test class is a class with `TestAttributed` or `HasTestMethod`, or a non-abstract class whose `Base`, followed through the scanned sources by name, transitively reaches a test class.
- A test class is selected when its file is a seed file or its file's `Identifiers` name a type in the closure; a selected abstract test class selects every class deriving from it, transitively.
- Each selected non-abstract class gets `filterTerm`: `FullyQualifiedName~` followed by its namespace, its outer chain and its name joined by `.`, a nested class joined to its outer class by `+` (`FullyQualifiedName~Ns.Outer+Inner`), and an abstract class gets none.
- A selected class's project is the `.csproj` in the nearest ancestor directory of its file up to the `dir`; none there is `Whole` naming the file, and a nearest directory holding more than one `.csproj` is `Whole` naming the directory.
- A non-empty seed that selects no class sets `NoTestClass`; an empty seed is the empty scope.

`doc.go` gains the closure, the test class and selection rules, the term spelling, the project rule and the Bound of Decision 2: the closure misses only references that never spell the type's name, which the whole gates catch.

Tests:
`TestNarrow` is a `tier0` table over sources built with `parseSource` from in-memory files and a project list, asserting the `Scope` exactly:
a changed production type selects the test class naming it and the one naming a type that names it (transitive), and leaves an unrelated test class out;
a changed test file is selected though it names no changed type;
an attribute-free fixture deriving from an abstract base with `[Test]` methods is selected through the base, with its own term, and the base gets none;
a nested test class gets the `+` term;
two test projects each get their own `Project`;
a seed file with no `.csproj` above it, a selected class with none, and a nearest directory with two `.csproj` files each yield `Whole` with their reason;
a seed that selects nothing sets `NoTestClass`, and an empty seed yields the empty scope.

**ImpactSummary:** new members of the package card 4 started; only its own doc grows.
