# Card 16 — hubforge-templates

**Create:**
- `plan:internal/hubforge#Shape` -> `type Shape struct`
- `plan:internal/hubforge#Shapes` -> `func Shapes() []Shape`
- `plan:internal/hubforge#TemplatePairSlug` -> `const TemplatePairSlug = "pair"`
- `plan:internal/hubforge#templatePrefix` -> `const templatePrefix = "hubforge-template-"`
- `plan:internal/hubforge#hubTemplate` -> `type hubTemplate struct`
- `plan:internal/hubforge#templates` -> `var templates map[string]*hubTemplate`
- `plan:internal/hubforge#templatesMu` -> `var templatesMu sync.Mutex`
- `plan:internal/hubforge#templateFor` -> `func templateFor(tb testing.TB, shape Shape) *hubTemplate`
- `plan:internal/hubforge#CopyHub` -> `func CopyHub(tb testing.TB, shape Shape) *Hub`
- `plan:internal/hubforge#SharedHub` -> `func SharedHub(tb testing.TB, shape Shape) *Hub`
- `plan:internal/hubforge#relocateHub` -> `func relocateHub(src, dst string) error`
- `plan:internal/hubforge#fingerprint` -> `func fingerprint(root string) (string, error)`
- `plan:internal/hubforge#hubAt` -> `func hubAt(tb testing.TB, root string, shape Shape) *Hub`

**Edit:**
- `internal/hubforge#NewHub`
- `internal/hubforge/hub_test.go`
- `cmd/lyx/tierpurity_test.go`
- `cmd/lyx/hermeticenv_test.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:internal/hubforge#markFixtureBuild`
- `internal/fslink/fslink.go`

**Prosa:**
- `internal/hubforge/doc.go`
- `pattern/PATTERN-hubforge-fixtures.md`
- `pattern/PATTERN-test-speed.md`

**Intent:** D11: hub fixtures are templates, copies and a shared read-only hub.
`Shape` is an anchor plus an ordered list of pair slugs; `Shapes` is the closed set: `.` and `backend`, each with no pair and with the one pair `TemplatePairSlug`.
`templateFor` builds, once per test binary and per shape under `templatesMu`, a template hub through `NewHub` plus `AddPair` for the shape's pairs, under `markFixtureBuild`, in an `os.MkdirTemp` directory whose last element begins with `templatePrefix`, which no `t.TempDir` path element carries, so no copy path has the template root as a string prefix; it records the build-time `fingerprint`, the sorted list of every regular file's relative path, size and mtime, and a `poisoned` flag.
A shape outside the set is a `tb.Fatalf` naming `NewHub` as the way forward; `NewHub` stays for such shapes and gains a doc sentence pointing at the two new entry points.
`CopyHub` checks the template's fingerprint against the build-time one, failing the test naming the fixture on a mismatch or a poisoned template, then `relocateHub` copies the template into the test's own `tb.TempDir`: every regular file copied without following links, the template root path rewritten to the copy's in every text file in both separator spellings, every link read through `fslink.PointsTo` and recreated through `fslink.CreateDirLink` with its rewritten target, the bares copied and their URLs rewritten the same way, and the copy verified to contain the template root nowhere before `hubAt` builds the `Hub` for it through `lyxcwd.Resolve`, registering the junction teardown `NewHub` registers.
`SharedHub` runs the same entry check and returns the template's own `Hub`, with a cleanup that fingerprints the tree in-process and, on a change, marks the template poisoned and fails the test naming the fixture and saying the blame may land on a parallel sharer.
A write is anything that may change a file under the hub: a fabric mutating verb, `SeedConfig`, `AddPair`, any run of the `lyx` binary, any in-process CLI verb, and any spawned git subcommand; a read is a file read, a link read or an in-process go-git read.
The tier-purity scan's `bannedTokens` and the hermetic scan's `gitSpawnTokens` gain `hubforge.CopyHub` and `hubforge.SharedHub`, since both build a real hub on first use.
Tests, integration-tagged in `hub_test.go`: `TestCopyHub` on the `.` one-pair shape asserts `fabric status` is clean through `fabriccli.RunCLIIn` on the copy's prime, `git worktree list` resolves every worktree, the pair's portal link and launcher directory resolve, no file under the copy contains the template root, and `assertRealHub` holds; `TestSharedHub` asserts a test that writes a file into the shared hub fails at its cleanup naming the fixture (through a recording `testing.TB`), and that a later `SharedHub` and `CopyHub` of the same shape in the binary fail naming the fixture; `TestBuildBareTemplate` is unchanged.
`hubforge`'s budget row rises by two.
The package doc describes the template-and-copy model as extended to the hub, the relocation, the fingerprint and the poisoning; `PATTERN-hubforge-fixtures.md` says a copy is a relocation of a hub `fabriccli.CloneAndWire` built; `PATTERN-test-speed.md`'s "untagged tests spawn nothing" lists the two new tokens.

**ImpactSummary:** `NewHub` gains a doc sentence only; two scans gain two tokens each; every later hub-taking test can move onto the two new entry points without `NewHub` changing shape.

**Verify:** lyx gate test --tags integration ./internal/hubforge
