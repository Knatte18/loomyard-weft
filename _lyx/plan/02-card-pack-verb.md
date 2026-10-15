# Card 2 — card-pack-verb

**Create:**
- `plan:internal/websterengine#CardPackMarkdown` -> `func CardPackMarkdown(index planindex.Index, planDir, worktreeRoot, stencilsDir string, number int) ([]byte, error)`
- `plan:internal/loomcli#loomCLI.cardPackCmd` -> `func (c *loomCLI) cardPackCmd() *cobra.Command`
- `internal/loomcli/cardpack_test.go`

**Edit:**
- `internal/loomcli#Command`
- `internal/loomcli#verbUsesLightweightWiring`
- `internal/loomcli#loomCLI.wireLightweight`
- `internal/loomcli#loomCLI.resolveRunID`
- `internal/loomcli#TestVerbUsesLightweightWiring`
- `internal/loomcli#TestWireLightweight_FillsThePathsWithoutLoadingAnyConfig`
- `internal/loomcli#TestResolveRunID`
- `internal/loomcli/doc.go`
- `internal/websterengine/doc.go`
- `contracts/specs/refusal-spec.md`
- `docs/overview.md`
- `contracts/stencils/webster/webster-fragment-pack-card.md`
- `contracts/stencils/webster/webster-fragment-pack-unresolved.md`
- `contracts/stencils/webster/webster-fragment-pack-empty.md`
- `contracts/stencils#WebsterFragmentPackCard`

**Uses:**
- `internal/websterengine#renderCardPack`
- `internal/websterengine#cardPointerPath`
- `internal/websterengine#masterPlanDirDisplay`
- `internal/planindex#Index`

**Intent:**
Add the read-only verb `lyx loom card-pack <card>`, which renders one card's kick-start pack against the worktree as it stands now, committed or not (D3, D6).

`CardPackMarkdown`, in a new file `internal/websterengine/cardpack.go`, is the verb's whole logic, told every path and handed the index, so `websterengine` keeps importing neither `planglyph` nor quarry nor `lyxcwd`:
- It refuses a planDir that does not exist (`os.Stat` reports not-exist), then parses the plan afresh with `planparser.ParsePlan(planDir)` and refuses a parse error, then refuses a number no card of `plan.Cards` carries.
  Each refusal names the plan directory or the card number and ends with the way forward `way forward: locate the card's glyphs with "lyx quarry resolve" or a grep`, the per-card prompt's fallback.
  Any other stat error, and a failed read of a pack fragment, is returned wrapped, the raw I/O transient of the loom section, whose Trigger this card extends.
- It calls `index.KickstartPack(plan, []planparser.Card{card}, worktreeRoot)` and renders the one card through `renderCardPack` under the heading `cardPointerPath(card, masterPlanDirDisplay(worktreeRoot, planDir))`, so the subsection is byte-identical to that card's subsection in the batch pack: the same `webster-fragment-pack-card`, `-pack-unresolved` and `-pack-empty` fragments, read from stencilsDir.
  A nil result (a glyph-less language) renders the card with an empty `planindex.Pack`, which prints the empty-card line;
  so does a card naming no member glyph, whose `plan:` handles still list as unresolved.
  A `plan:` handle stays unresolved, since handles bind at record-batch.
- An index error is a Warn, `webster: card pack index failed, keeping the degraded pack`, with card and error, and the degraded pack the index returned is rendered: every glyph unresolved with the error, no refusal.
- Once a pack is rendered it logs one Info line, `webster: card pack`, with keys card, resolved and unresolved, counted as distinct refs exactly as `buildKickstartPack`'s line counts them.
  A refusal returns before that line, so a refused call logs none.
- It returns the rendered subsection with one trailing newline.

`cardPackCmd`, in a new file `internal/loomcli/cardpack.go`, is `Use: "card-pack <card>"`, `Args: cobra.ExactArgs(1)`, with the `clihelp.AudienceRole` annotation, since a fork's prompt names it, a lower-case `Short` and a `Long` within the help walk's limit that says it prints one card's pack as plain markdown and reads no webster state.
It refuses an argument that `strconv.Atoi` rejects or that is `0` with the standard failure envelope (`output.Err`), naming the argument and the same way forward, and exit 1.
A leading `-`, as in `-1`, is taken by cobra as a flag and refused by cobra's own flag parse before `RunE` runs, as is a call with no argument or two by `cobra.ExactArgs(1)`, so neither reaches this refusal.
It calls `CardPackMarkdown` with `c.env.PlanIndex`, `planparser.PlanDir(c.env.AnchorPath)`, `c.env.AnchorPath` (the root begin-batch's pack resolves against, `hubgeom.WebsterGeometry`'s WorktreeRoot) and `c.env.StencilsDir`;
an error goes out through `output.Err` with exit 1, and success writes the markdown to stdout verbatim, exit 0, with no envelope, as `lyx quarry resolve` writes its own answer.
`Command` registers it beside `lintPatternCmd`.
`verbUsesLightweightWiring` admits `card-pack`, since it reads only path fields and the index, and its doc comment names the verb with the other read-only verbs.
`wireLightweight` fills `c.env.StencilsDir` with `fabricengine.StencilsDir(location.HubPath)`, the value `hubgeom.WebsterGeometry` gives webster.
`resolveRunID` treats card-pack's positional argument as a card number, never a run-id, as it does insert-card's.
The verb resolves its location through `lyxcwd` in loom's pre-run like every other loom verb: every loom verb is hub-only, so `internal/cliwire`, the standalone-capable wiring the record's constraint names, does not apply, as `internal/loomcli/doc.go` says of the peers group.
Loom's pre-run only needs a git repository at its anchored directory, so a call from a standalone target passes it, and the verb refuses on the missing plan directory, since a standalone run's plan lives in its state directory and not under the anchor;
its way forward is the locate fallback, so a standalone webster run never gets a working pack from the verb and card 3's begin-batch renders the file or inline layout there.

Bound: the verb mutates no webster, plan or tracked state and holds no lock, so it adds no route around any gate.
Beyond its stdout it writes only through `internal/logger`, as every lyx verb that logs does: the process's own trace file under the anchor's `.lyx/logs`, the Info line, the Warn on an index failure, which also reaches stderr at the logger's default threshold, and the exit retention sweep.
It is a `lyx loom` verb rather than a `lyx webster` one because the claudeengine hook refuses every `lyx webster` command from a fork; the hook is untouched.

Tests, all in the new `cardpack_test.go`, one table-driven `TestCardPackCmd` over a hand-populated `*loomCLI` (`AnchorPath`, `WorktreeRoot`, `StencilsDir` from `stencilkit.Seed`, `PlanIndex`), its plan written with `plankit.Write` under `<anchor>/_lyx/plan`, its log captured with `logcapture.CaptureVerbose`, so it does not call `t.Parallel`:
- a card with an Edit glyph `internal/foo#Baz` and a `Uses:` glyph that does not resolve, over a Go file holding `Baz`, with the real `planglyph.NewIndex`: exit 0, stdout equals the exact subsection, heading `### `_lyx/plan/01-<slug>.md``, the `Baz` entry with its span and signature and the `#### Unresolved` list, and the log holds `msg="webster: card pack"` with `card=1 resolved=1 unresolved=1`;
- the same fixture after an uncommitted edit inserts lines above `Baz`: the span in stdout is the shifted current one, not the first call's;
- a `go` plan whose card targets only a plain path and a `plan:` handle: exit 0, stdout holds `no member glyphs on this card` and a `#### Unresolved` list naming the handle, and the log holds the Info line with `card=1 resolved=0 unresolved=1`;
- a `language: none` plan whose card targets only a plain path: exit 0, stdout holds `no member glyphs on this card` and no `#### Unresolved` heading, and the log holds the Info line with `resolved=0 unresolved=0`;
- a test-local index embedding `planindex.Index` whose `KickstartPack` returns the card with its glyph unresolved and an error wrapping `planindex.ErrQuarryUnavailable`: exit 0, stdout lists the glyph unresolved with that error, the log holds a `level=WARN` line and the Info line;
- refusals, each asserting exit 1, a failure envelope whose error ends with the way forward, and no `webster: card pack` line in the log: argument `x`, argument `0`, no argument (cobra's `ExactArgs` error, no envelope asserted), card `9` of a one-card plan, an anchor with no `_lyx/plan` directory, and a plan whose Card Index names a card with no card file.
`TestVerbUsesLightweightWiring` gains the row `{"CardPack", "card-pack", true}`;
`TestWireLightweight_FillsThePathsWithoutLoadingAnyConfig` asserts `c.env.StencilsDir` equals `fabricengine.StencilsDir(location.HubPath)`;
`TestResolveRunID` gains "card-pack's card number is not a run-id", args `["3"]`, wanting `shedrun.SelfRunID`.
`internal/loomcli` stays within its row in `cmd/lyx/testdata/test-budget.yaml`.

Docs in this commit: `contracts/specs/refusal-spec.md`'s loom section gains one row per refusal (an argument that is not an integer or is `0`, unknown card number, missing plan directory, plan that does not parse), each a correctness halt with the way forward above, and its grouped raw I/O row's Trigger gains the plan directory's stat and the pack fragment reads;
the banners of `webster-fragment-pack-card`, `-pack-unresolved` and `-pack-empty` name `lyx loom card-pack`'s stdout as a third destination beside the pack file and the inline section, and the `WebsterFragmentPackCard` doc comment also names rendering one card alone as `lyx loom card-pack`'s output;
`internal/loomcli/doc.go` gains a `# Card pack verb` section naming the verb, its output and its bound;
`internal/websterengine/doc.go`'s kick-start pack section describes `CardPackMarkdown` and the `webster: card pack` Info line and its keys;
`docs/overview.md`'s loom module line lists `card-pack`, and its loom verb paragraph gains one line saying what `card-pack <card>` prints.

**ImpactSummary:** a new role verb under `lyx loom` and an exported engine function; lightweight wiring now fills StencilsDir, and the wiring, run-id and docs tables gain one entry each, with no existing verb's behavior changed.

**Commit:** 2: loom: card-pack verb renders one card's kick-start pack against the current tree
