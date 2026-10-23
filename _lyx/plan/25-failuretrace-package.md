# Card 25 — failuretrace: from a failed verify log to plan cards

**Create:**
- `internal/failuretrace/doc.go`
- `internal/failuretrace/testmain_test.go`
- `plan:internal/failuretrace#Placement` -> `type Placement struct`
- `plan:internal/failuretrace#Locations` -> `func Locations(failures []websterengine.VerifyFailure, p Placement) (placed, untraced []string)`
- `plan:internal/failuretrace#Input` -> `type Input struct`
- `plan:internal/failuretrace#Trace` -> `func Trace(in Input) (verifytree.FailureTrace, error)`
- `plan:internal/failuretrace#NewTracer` -> `func NewTracer(worktreeRoot, anchorPath string) func(log, head string) (verifytree.FailureTrace, error)`
- `plan:internal/failuretrace#TestLocations` -> `func TestLocations(t *testing.T)`
- `plan:internal/failuretrace#TestTrace` -> `func TestTrace(t *testing.T)`

**Edit:**
- `docs/overview.md`
- `cmd/lyx#treeSitterAllowedLinks`

**Uses:**
- `internal/websterengine#ParseVerifyFailures`
- `internal/websterengine#State`
- `internal/websterengine#BatchState`
- `plan:internal/planparser#TargetHolds`
- `internal/planparser#RefMember`
- `plan:internal/planglyph#EncloseAt`
- `internal/planglyph#Resolve`
- `internal/planglyph#Delta`
- `plan:internal/verifytree#FailureTrace`
- `plan:internal/verifytree#TracedCard`
- `plan:internal/verifytree#CommitRange`

**Intent:**
D9: a new package, `internal/failuretrace`, turns a failed Merge-In or Publish verify log into the record's `cards`, `later_range` and `untraced`.
It lives apart from `landingshed` and `loomshed`, which link no tree-sitter grammar, because it reaches quarry through `internal/planglyph`; it is told absolute paths and derives none (PATTERN-told-geometry), reads plan refs only through `internal/planparser`, and carries the package's `testmain_test.go` as its siblings do.
`doc.go` holds the design below, the location cap and the open risk that the trace often names no card.
`docs/overview.md`'s tree of internal packages gains `failuretrace` with a one-line purpose.
`treeSitterAllowedLinks` gains `{Key: "internal/failuretrace", Why: "traces a failed verify through planglyph's EncloseAt"}`, since the guard in `cmd/lyx/treesitterlink_integration_test.go` fails any test binary outside it that links tree-sitter.

`Locations(failures, p)` extracts `path:line` locations, Go test and build output only, from each failure identity's `Tail`: tokens `<path>.go:<line>` with an optional `:<col>`, which is dropped.
`Placement` holds `WorktreeRoot`, `ModulePath` (from the worktree's `go.mod`, empty when unknown) and `ExistsAtHead func(rel string) bool`.
Each location is placed relative to the repo root: a path that exists at the head as written; an absolute path under the worktree with the root stripped; a path opening with the module path and a slash with that prefix stripped, which places the panic frames `-trimpath` writes (the hub's gate tiers set it, `trimpathLine` in `internal/gatecli/tiers.go`); a bare file name or a `./`-relative one joined onto its identity's package directory, the import path less the module path.
An opaque identity carries no package, so its bare or `./`-relative locations go to `untraced` as written; any other location, such as a standard-library or dependency frame, is dropped.
Placed locations are deduplicated in first-seen order and capped at 20 (the auto-mode assumption).

`Input` holds the log, the record's `Head`, the `Placement`, the parsed live plan, Webster's `*websterengine.State`, and three quarry seams: `Enclose func(rev string, targets []string) ([]quarry.EncloseResult, error)`, `Resolve func(targets []string) ([]quarry.ResolveResult, error)` and `Delta func(from, to string) (quarry.GitDeltaAnswer, error)`.
`Trace(in)`:
1. places the locations of `ParseVerifyFailures(in.Log)` and calls `Enclose(in.Head, placed)`;
2. matches a found answer's member glyphs to every live card with a target that `planparser.TargetHolds` (glyph, containing type, unit self glyph, file);
3. matches a `not_found` answer, a location inside no member such as an unused import, through its `Unit`, the file's self glyph: every live card with a target holding that file or its self glyph, or a member-glyph target (through `RefMember`, so a `plan:` handle is its glyph) that resolves into that file at `Head` through `Resolve`; a member that does not resolve there, such as one its card deletes, renames or moves out, is looked up in `Delta(batch start, Head)`'s deleted, renamed and modified symbols' before-side files, and one found in neither matches only by the file and unit rule;
4. puts in `untraced` a rejected location, a `not_found` location matching no card and a found location no live card's targets hold, each as `path:line`, after the opaque identities' locations;
5. reads each matched card's batch from `State.Batches` by the live card id `NN-<slug>`, the number zero-padded as begin-batch records it (`%02d-%s`, never `Card.ID()`'s unpadded form), a batch whose record has no `Cards` reading as the single card `NN-<Slug>` of its own number, as `BatchState.Cards` documents; its range from the batch's `StartSHA` to the last element of its `CardSHAs` (empty for a card Webster holds no record of, or a batch with no `CardSHAs` element), and the later range from the recorded head of the highest-numbered matched batch that has one to `Head` (empty when none has one).
Cards come out in card-number order, each with its matched glyphs and locations.
A seam error is returned; so is a nil plan or nil state, since the caller logs the failure and records no trace.

`NewTracer(worktreeRoot, anchorPath)` returns the seam landingshed is told: on each call it parses the plan under the anchor, loads Webster's state (`websterengine.LoadState` over the anchor's directories), reads the module path from `go.mod`, and wires `planglyph.EncloseAt`, `planglyph.Resolve` and `planglyph.Delta` over `worktreeRoot`; `ExistsAtHead` stats the worktree, which the verify ran on clean at `Head`.
A missing plan or state is an error.

Tests, in package `failuretrace`, with fakes for the three seams:
`TestLocations` rows: a test failure's bare `x_test.go:12` placed through its package; a build error `internal/x/y.go:3:5` placed as written; an absolute worktree path stripped; a `-trimpath` panic frame `<module>/internal/x/y.go:9` placed repo-relative; a `./y.go:4` under an opaque identity left untraced; a standard-library frame dropped; 25 distinct locations capped at 20.
`TestTrace` rows: a build error inside a member a card targets names that card, its batch and range; an unused-import `not_found` in a file a card targets names it, and so does one in a file where a card targets only a member glyph that resolves there; a member target that resolves only at the batch start through `Delta` matches; a failing test a card creates under a `plan:` handle names that card; a batch with no `CardSHAs` element gives an empty range and, when it is the only matched batch, an empty later range; a card Webster holds no record of gets batch 0 and an empty range; a card numbered below 10 is found under its padded id, and one in a batch record without `Cards` is found through the batch's own number and slug; a member target that resolves neither at `Head` nor in `Delta` matches only by the file and unit rule; a done batch followed by later commits gives the later range from its head to `Head`; a nil plan and a nil state are each a returned error; a `.md` location, a location no card's targets hold and an opaque bare name land in `untraced` as `path:line`; an `Enclose` error is returned.

**ImpactSummary:** a new package with no caller until card 26; `docs/overview.md` gains one line in its package tree, and the tree-sitter link guard's list one entry.
