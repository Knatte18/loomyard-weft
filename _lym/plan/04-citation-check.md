# Card 4 — citation-check

**Create:**
- `plan:internal/loomshed#citationFindings` -> `func citationFindings(worktreeRoot string, index planindex.Index, citations []discussionparser.Citation) ([]discussionparser.Finding, error)`
- `plan:internal/loomshed#goPackageDirs` -> `func goPackageDirs(worktreeRoot string) ([]string, error)`
- `plan:internal/loomshed#TestValidateLoomDiscussion_Citations` -> `func TestValidateLoomDiscussion_Citations(t *testing.T)`
- `plan:internal/loomshed#unavailableIndex` -> `type unavailableIndex struct`
- `plan:internal/testkit/envkit#noFindingsIndex.ResolveGlyphs` -> `func (noFindingsIndex) ResolveGlyphs(worktreeRoot string, glyphs []string) ([]planindex.GlyphAnswer, error)`

**Edit:**
- `internal/loomshed#ValidateLoomDiscussion` -> `func ValidateLoomDiscussion(decisionRecordPath, supportLogPath, relevantPath, worktreeRoot string, index planindex.Index) ([]discussionparser.Finding, error)`
- `internal/loomshed#NewDiscussionGate` -> `func NewDiscussionGate(decisionRecordPath, supportLogPath, relevantPath, worktreeRoot string, index planindex.Index) shuttleengine.Gate`
- `internal/loomshed#loomshedAllowedImports`
- `internal/loomshed#TestNewDiscussionGate`
- `internal/loomshed#TestDiscussionGate_FailureSurfacesItsFindings`
- `internal/shedrecipe#resolvePlainGateClosure`
- `internal/shedrecipe#resolveGateSpec`
- `internal/shedrecipe#Env`
- `internal/shedrecipe#TestResolveGateSpec_ValidOneEntryPerVocabularyName`
- `internal/shedrecipe#TestResolveGateSpec_Rejections`
- `internal/loomcli#loomCLI.validateDiscussionCmd`
- `internal/loomcli#TestGateParity_DiscussionGate`
- `internal/loomcli#TestSmokeGate_RepromptsThroughARealPaneAndFixesTheArtifact`

**Extends:**
- `contracts/specs/loom-discussion-spec.md`
- `contracts/stencils/loom/loom-template-discussion.md`
- `internal/loomshed/doc.go`
- `internal/loomcli/doc.go`

**Uses:**
- `plan:internal/discussionparser#Citations`
- `plan:internal/discussionparser#Citation`
- `plan:internal/planindex#GlyphAnswer`
- `internal/planindex#ErrQuarryUnavailable`
- `internal/gitrepo#New`
- `internal/gitrepo#Repo.CurrentSHA`
- `internal/gitrepo#Repo.SubdirsAtRevision`
- `internal/gitrepo#Repo.FilesInDirAtRevision`
- `internal/lymdirs#LymDirName`
- `internal/lymdirs#DotLymDirName`
- `internal/gitkit#CopyRepo`
- `internal/gitkit#Git`
- `internal/loomshed#validDecisionRecord`
- `internal/loomshed#writeDiscussionFixture`

**Intent:**
D3 and D4: on a loom run the decision record's cited paths and symbols must exist.

`ValidateLoomDiscussion` gains `worktreeRoot` and `index` and runs, after `CheckDecisionIDs` and before `CheckRelevant`, `citationFindings` over `discussionparser.Citations` of the record it already read; a record `Validate` reports missing adds nothing.
`ValidateDarnDiscussion` is unchanged: a darn record's citations are never tested.
`NewDiscussionGate` gains the same two parameters and passes them through.
`resolvePlainGateClosure`'s loom form of "discussion" additionally requires `env.WorktreeRoot` through `requireAbsRoot` and `env.PlanIndex` through `requireSeam`, so a loom recipe wired without them fails loud at resolve time, and `resolveGateSpec`'s doc comment says so; the darn form requires neither.
`validateDiscussionCmd` passes `c.env.WorktreeRoot` and `c.env.PlanIndex`, which `wireLightweight` already fills.
The `shedrecipe.Env` field docs of `WorktreeRoot` and `PlanIndex` name the loom "discussion" gate among their readers.
`envkit`'s `noFindingsIndex` gains `ResolveGlyphs`, answering every glyph `GlyphFound` with no files, so a gate built from `FullEnv` that reaches the citation or plan card checks never calls the nil embedded index.

`citationFindings`, in a new file of `internal/loomshed`, skips every marked citation and every token holding whitespace or `<`, then classifies each remaining token, in this order:

- Path: the token holds `/`, none of `#`, `>`, `*`, `{`, `}`, `$`, `:`, no `...` segment, and its first segment is a root entry git tracks: one of the directory or file names `gitrepo.New(worktreeRoot)` lists directly at the root of HEAD's tree (`CurrentSHA`, then `SubdirsAtRevision` and `FilesInDirAtRevision` at `""`), other than `lymdirs.LymDirName` and `lymdirs.DotLymDirName`. The tracked roots are read once per call and only when some token is path-shaped; a worktree with no commit has no tracked root, so no path is tested there, and any other read error is a returned error.
Reading HEAD's tree is a narrowing of "an entry git tracks", stated as a bound: a root entry staged but never committed is not tested. The path must exist under `worktreeRoot` (`os.Stat`), and a token ending in `/` must be a directory; otherwise one `citation-path-missing` finding naming the token and its line, with "way forward: correct the path, or mark one the task creates or removes with ` (new)` or ` (removed)` right after the closing backtick".
- Glyph: the token is not `plan:`-prefixed and parses under the Go glyph grammar (`glyph.Parse` of `github.com/Knatte18/quarry/glyph`) as a member glyph `<unit>#<member>` whose unit holds `/` or `.` and does not name an existing worktree file other than a `.go` or `.cs` file. So `## Decisions`, `C#`, `#123` and `docs/overview.md#Modules` are no glyph.
- Qualified symbol: the token, with one trailing `()` removed, has no `/` and is `pkg.Member` or `pkg.Member.Sub…`, where `Member` holds an upper-case letter and `pkg` is the base name of a directory `goPackageDirs` returns. With exactly one such directory it is resolved as the glyph `<dir>#Member`, so `shedrecipe.Env.PlanIndex` resolves as `internal/shedrecipe#Env`; with two or more it is one `citation-symbol-ambiguous` finding naming the directories, with "way forward: spell the symbol as a glyph". `state.json` (lower-case member), `os.ReadFile` (no worktree package `os`) and an unqualified name are no symbol.

Every glyph and qualified symbol of the record is resolved in one `index.ResolveGlyphs` call, made only when there is at least one; its error is returned as is, so a quarry outage wrapping `planindex.ErrQuarryUnavailable` is never a pass and never a finding.
A `GlyphMissing` answer is one `citation-symbol-missing` finding naming the token and line, with "way forward: correct it as `lym quarry glyphs <dir>` lists the members, or mark it ` (new)` or ` (removed)` right after the closing backtick"; a `GlyphAmbiguous` answer is one `citation-symbol-ambiguous` finding naming the candidates' files, with "way forward: cite the declaring file meant by its path in place of the symbol".
Findings carry the record's path and keep record order.

`goPackageDirs` walks `worktreeRoot` and returns, sorted and slash-separated, every directory holding a `.go` file, skipping a directory named `testdata` and any whose name starts with `_` or `.`; it is computed once per call and only when a qualified-symbol candidate exists.

`loomshedAllowedImports` gains `github.com/Knatte18/quarry/glyph`, with a comment that it is the cgo-free glyph grammar `internal/planparser` already imports, and `internal/lymdirs`, with a comment that it names the two root directory names and resolves no geometry; `TestToldGeometryInvariant_AllowlistOnly` then passes.

Bound, stated in the spec: a marked token or one holding `<` is not tested, so a writer can exempt a wrong claim that way and review still reads it; a path outside backticks, a slash-free file name, a package pattern with `...`, a token under an untracked root entry and run-state paths under `_lym/` or `.lym/` are not tested; a member after the first of `pkg.Member.Sub` is not tested; the Discussion-Write and Discussion-Burler-Fix gates now halt on a quarry outage as an infrastructure fault.

Tests:
`TestValidateLoomDiscussion_Citations`, tier1, a table in a new file `internal/loomshed/citations_integration_test.go` with the `//go:build tier1` line, since it commits a git fixture: a repository from `gitkit.CopyRepo` with `internal/real/real.go` declaring `type Thing struct{ Field int }` and `func Do()`, `internal/real/twice_a.go` and `internal/real/twice_b.go` both declaring `func Twice()` under no build constraint, `internal/dup/a/x/x.go` and `internal/dup/b/x/x.go` both declaring package `x`, `internal/real/testdata/real/real.go`, `docs/overview.md` and `_lym/keep.md`, committed with `gitkit.Git`, and a told record per row with all required sections.
Rows: `internal/missing/file.go` fails with `citation-path-missing` naming the token, its line and the way forward; the same token marked ` (new)` passes; `github.com/Knatte18/quarry/glyph`, `_lym/plan/missing.md` (under a tracked `_lym` root, so only the `lymdirs` exclusion passes it), `internal/<name>/x.go` and a missing path inside a fenced block pass; `internal/real/` passes and `internal/real/real.go/` fails; `internal/real#Nope` and `real.Nope` fail with `citation-symbol-missing` and its way forward; `internal/real#Thing`, `real.Thing`, `real.Do()` and `real.Thing.Field` pass; `x.Foo` yields `citation-symbol-ambiguous` naming both directories and the "spell the symbol as a glyph" way forward, while `real.Thing` stays unambiguous although `testdata/real` exists; `internal/real#Twice` yields `citation-symbol-ambiguous` naming `internal/real/twice_a.go` and `internal/real/twice_b.go` and the "cite the declaring file" way forward; `## Decisions`, `C#`, `docs/overview.md#Modules`, `state.json` and `os.ReadFile` pass untested; a missing path inside an `### Added after Discussion` entry, on a suffixed PATTERN changes bullet and inside a carry-over block passes; a worktree with no commit and a record citing `internal/missing/file.go` passes; a record citing `internal/real#Thing` told `unavailableIndex`, a fake embedding `planindex.Index` whose `ResolveGlyphs` returns an error wrapping `planindex.ErrQuarryUnavailable`, makes `ValidateLoomDiscussion` return that error and no findings; a darn record citing a missing path passes `ValidateDarnDiscussion`.
`TestNewDiscussionGate`, `TestDiscussionGate_FailureSurfacesItsFindings`, `TestGateParity_DiscussionGate` and `TestSmokeGate_RepromptsThroughARealPaneAndFixesTheArtifact` pass the worktree root and a `planglyph.NewIndex(fabricengine.NewReferenceRule())` index to the re-signed functions; their records cite no path or symbol, so their verdicts are unchanged.
`TestResolveGateSpec_Rejections` gains two loom-form "discussion" rows: a nil `PlanIndex` and a relative `WorktreeRoot` each fail resolve naming the field.
`TestResolveGateSpec_ValidOneEntryPerVocabularyName`'s darn-form "discussion" step gains a case: with `DiscussionHistory` set, a nil `PlanIndex` and an empty `WorktreeRoot` still resolve.

Docs: `contracts/specs/loom-discussion-spec.md` gains a citations subsection for loom records: which tokens are tested as paths and as symbols, the ` (new)` and ` (removed)` marks, the `<placeholder>` rule for a name that must not exist, the parts no writer authors that are not tested, the bound above, and that a Go symbol is cited qualified with its package or as a glyph.
`contracts/stencils/loom/loom-template-discussion.md` gains, in its self-check step, one sentence saying the gate runs the decision-id, acceptance and citation checks the discussion spec's decision record section states and pointing there, restating no rule; the chair stencil gets it through its include, and the darn writer stencil, which already points at the spec, gains nothing.
`internal/loomshed/doc.go` and `internal/loomcli/doc.go` name the citation check of the loom discussion check, and `internal/loomshed/doc.go`'s sentence that the plan gates resolve through a told `planindex.Index`, so the package links no tree-sitter grammar, widens to the discussion gate's citation check and names the cgo-free `quarry/glyph` grammar it parses with, and `validateDiscussionCmd`'s Long help names it, on a loom run only.

**ImpactSummary:** two loomshed signatures gain the worktree root and index; their callers are the shedrecipe "discussion" gate resolver, the `validate-discussion` verb and four tests, all listed, and the loom discussion gates now depend on quarry.

**Commit:** `4: Check the decision record's cited paths and symbols on a loom run`
