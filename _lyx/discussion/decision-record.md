# Decision record — loom-stencil-wording

## Goal

Fix the loom and shed stencil wording the stencil intent audit of 2026-10-15 found wrong, and stop the hub-generic plan, Plan-Review and webster implementer stencils from naming this repository's Go verify tiers, which a hub of another language would otherwise receive.

## Scope

In:

- Rejection findings as a reviewer's claims (D1).
- F15 driver wording (D2), F16 advisor wording (D3), F17 plan wording and the same rework gap (D4).
- The `verify_tiers` key in `language.yaml` (D5), rendered at the sites D6 names, the deployed spec `contracts/specs/loom-plan-spec.md`'s `## verify model` among them.
- The docs for the new key: `internal/languageconfig/doc.go`, the template's key comment, and the doc of each package whose prompt or rubric fill now carries the key.

Out:

- The Go-coded mechanics that stay Go whatever the hub: `lyx gate test`, the card and batch gates `internal/websterengine` derives, the shell guard's raw-go row in `internal/shellguard`, and the plan gate's `verify-module-wide` and `verify-nested-module` checks.
  The stencil sentences that describe what those refuse stay.
- The gate commands and machine-run tier config `dotnet-verify-protocol` and `dotnet-scoped-verify` ask for: `verify_tiers` is prompt guidance only.
- The orch role stencil (`orch-role-stencil-fixes`) and the webster fork and pattern stencils (`webster-fork-stencil-fixes`).
- `lyx loom reject`'s help text.
- Setting this hub's own `verify_tiers` value (D7).

## Decisions

### D1 — Rejection findings are a reviewer's claims

**Decision:** `loom-template-rework.md` calls the rejection's `findings` field the findings of the reviewer that rejected the pull request, the alignment agent or the parent's review, and tells the writer to weigh them as a review against the code and the decision record, never as an operator ruling.
The rework stencil names one case of the plan stencil's late scope-addition route for itself: a finding that would overturn a Decision of the record is planned only once the writer has asked its parent to record it with `lyx loom decision add` and the record carries it.
If the parent declines, or no parent is recorded, the writer stops the whole rework: it plans no further finding, writes no `coverage.md`, and ends its session reporting the finding, so the seat's missing output halts the run for the parent.
The rework stencil also states that a finding that overturns no Decision is not a late scope addition under the plan stencil's route and is planned directly, licensing its card's scope;
this is the rework stencil's own exception to the plan stencil, which its Step 1 allows, so it points at the route without restating it.
Every other finding, one that adds scope the record does not cover included, is covered by a card with no recording and licenses that card's scope, as today.
The Plan-Review rubric's rework answer-key bullet says the same: a finding is a reviewer's claim the rework plan answers, it licenses a card's scope unless it overturns a Decision, and a card carrying a finding that overturns a Decision with no `Added after Discussion` entry recording it is a finding.
**Rationale:** a PR rejection comes from the alignment agent's report (`applyReject` in `internal/landingshed/pralign.go`) or the orch's reviewer fork, and operator rulings live in the decision record; a writer or reviewer that weighs an agent's claim as a ruling lets it override recorded decisions.
**Bound:** the route adds no verb and lets nothing new through; it holds back only Decision-overturning findings, each until the parent records it, and the whole-rework stop on a decline halts the run visibly instead of leaving a finding silently uncovered.

### D2 — F15: the driver names who fixes a halted run

**Decision:** the clause "so a fix committed by hand outside the run is pushed by the operator before `approve`, or goes through `reject` instead" becomes: a fix to a halted run is made by the run's own agents, through `reject` or a backward `goto`, which the driver reports and never runs, as `## Ways forward` already says.
**Rationale:** nobody commits by hand in a task pair, and the orch role forbids the orch to; the driver's `## Ways forward` already makes `goto` the parent's verb.

### D3 — F16: the advisor reads the draft record, not its format

**Decision:** the advisor stencil's role paragraph drops "read the record there for its format"; it names the record's path so the advisor reads the draft once it exists, to check it against PATTERN and the board, as its Step 3 says.
**Rationale:** the record does not exist until the chair writes it, and the advisor never writes one.

### D4 — F17: the plan and rework writers never ask the operator

**Decision:** `loom-template-plan.md`'s role line says the Plan producer never interviews and never asks the operator, and that its only messages are the parent requests its scope rules name.
`loom-template-rework.md`'s "You never interview and never ask" becomes: it never interviews and never asks the operator, and asks its parent as the parent directive says.
**Rationale:** both roles have parent requests their own text prescribes, and the parent directive makes the parent "the one to ask".

### D5 — `verify_tiers`: a hub-wide list in `language.yaml`

**Decision:** `language.yaml` gains a second key, `verify_tiers`: an ordered list of strings, each one rule of the hub's verify tiers (their order, which tiers are compiled rather than run and why, a nested-module form, the raw-run limits), written by the operator for the hub's language.
It is classified Guard like `skills` and stays out of the module's `LocalKeys`, so a local override cannot set it.
Its loader refuses what `skills` refuses (not a list of strings, an empty entry, surrounding whitespace) plus an entry holding a line break, with the existing way forward (`lyx config language` from the prime).
The template's value is the empty list; an absent key or an absent `language.yaml` resolves the empty list.
`languageconfig.Load` resolves both keys, so an invalid `verify_tiers` refuses every `language.yaml` consumer at spawn with that way forward, as an invalid `skills` does today.
**Rationale:** the tier order, tag names and compile-or-run rule are facts of the hub's language, and `language.yaml` is the hub-wide module that already holds the language-specific skills.
**Bound:** entries reach the plan writer, Plan-Review and implementer prompts verbatim, from a board-only file in the same trust class as `publish_verify`.
They cannot loosen a Go-coded guard: the shell guard and the plan gate still refuse what they refuse, whatever an entry says.
They can skip a tier: an entry calling a tier compiled rather than run lets the plan's `## verify:` section, which webster's plan-level verify gate runs as written, leave that tier unrun, and lets Plan-Review accept it.
The same section is the chained command Publish and Finalize run after a parent merge, so that tier also goes unrun at landing.
Who can write such an entry is bounded by the key's Guard classification on a board-only file.
Inside webster the skip is bounded by the Go-coded card and batch gates, which still run the untagged and `-tags integration` tests over the batch's packages whatever an entry says;
they run before the parent merge only, so at landing the skip is bounded by nothing but landing config's `publish_verify`, which runs only what that config names.
The `PATTERN-no-denied-recovery` scan does not cover config values, so an entry naming a denied form is the hub operator's error; the guard's refusal names it at run time.

### D6 — The stencils render `verify_tiers`; the Go text leaves them

**Decision:** the marker value is the entries as markdown bullets, rendered verbatim and never parsed as a template; every surrounding sentence, each empty-list fallback included, lives in the stencil.
At every site the marker is required and an empty list renders `none`, the way the rubric's notes do, so each fallback sentence reads beside the list or `none`.
Each site keeps only language-neutral rules:

- `loom-template-plan.md`'s `## verify:` paragraph: the section covers every targeted package and runs the hub's tiers in the order and form the list gives.
  With no list, it builds the code and then runs the tests of every package a card targets.
  The requirement to run each targeted package's tests, its hermetic build-tagged tests included, stays as a language-neutral rule, its `-tags integration` example dropped.
  The `go build`/`go vet`/`go test` order, the `llm` and `tmux` rules and the `go -C` form leave the stencil; the pointer to `verify-nested-module` stays.
- `loom-rubric-plan-review.md`'s `Verify coverage` bullet: a targeted package the section leaves unrun, its hermetic build-tagged tests included, is a finding, a tier the list says is compiled rather than run is not, and a section out of the list's order is a finding.
  With no list, only the coverage rule applies.
  The rubric is filled by `shedadapters.ReadRubric`, whose strict fill refuses an empty marker, so its fixed value map gains the key, and its callers (the Bouncer judge and seed rows and the Burler review row) are told the list.
- `webster-body-implementer.md`'s raw-run section, retitled to name raw test runs instead of `go` runs: a raw test run of the hub's language is allowed on the batch's listed packages only, and a tier the list says is compiled rather than run is compiled, never run, as the list says.
  The guard-describing sentence stays: a module-wide, `tmux`-tier or `llm`-tier run, which the shell guard refuses raw, goes through `lyx gate test`.
  The gate-describing sentence beside it stays on the same footing: a test under the `tmux` or `llm` tag that no gate step names is compiled, never run, through `lyx gate test --tags <tag> <packages> -- -run '^$'`;
  the list's compile-only rule adds tiers beyond those two.
  With no list, a raw run is still limited to the listed packages and to what that sentence allows.
  The sentence that a `lyx gate test` step runs as written, whatever its `--tags`, stays, so the list's compile-only rule governs raw runs and runs no gate step names, never a rendered card or batch gate step.
  The slot-busy exit text stays.
  Both value maps that fill the body, the fork's and the recovery strand's, carry the list.
- `loom-plan-spec.md`'s `## verify model`: the order sentence and the `tmux` sentence name `language.yaml`'s `verify_tiers` and landing config's `publish_verify` instead of Go commands.

This task edits none of the ranges `webster-fork-stencil-fixes` cites in the same files (the plan's `## Rename mechanic`, the implementer's failure and report sentences, the spec's `Rename` row and per-language list), so that task's sites move only by line drift.
**Rationale:** a .NET or Python hub then gets its own tiers, or a neutral rule, in its plan writer, Plan-Review and implementer prompts; the guard-describing sentences stay because the guards are installed in every hub.

### D7 — This hub's value is the operator's to set

**Decision:** `internal/languageconfig/doc.go` carries this hub's rules as its example of a Go hub's value:

- `go build ./...` first, then `go vet` under each of `-tags integration`, `-tags tmux` and `-tags llm`, then the untagged `go test`, then `go test -tags integration`.
- The `llm` and `tmux` tiers are compiled, never run: `llm` tests spawn a real LLM, and Publish runs the `tmux` tier through landing config's `publish_verify`; outside the plan's section a test of either is compiled through `lyx gate test --tags <tag> <packages> -- -run '^$'`.
- A nested module's packages are tested with `go -C <module>` and module-relative package paths.
- A raw `go test` runs only on named packages; a module-wide, `tmux`-tier or `llm`-tier run goes through `lyx gate test`.

The operator sets this hub's value from that example with `lyx config language` from the prime, in the editor.
**Rationale:** the hub-wide file lives on the board, outside every task worktree, so no card can set it; hub config is the operator's, and until it is set this hub's prompts fall back to the neutral rule, which the guards keep safe for a Go hub with the gaps Open risks names.
The editor avoids the quoting the entries' backticks and quotes would need on a `--set` line.

## Constraints

- `PATTERN-stencil-ownership`: every new or changed sentence lives in the stencils under `contracts/`; Go renders only the configured entries as bullets, or `none` for the empty list.
- `PATTERN-told-geometry` and `PATTERN-config-layers`: `languageconfig` reads the file through `configengine`, the CLI wiring resolves it, and the engines and rubric callers are told the list.
- `PATTERN-hub-wide-config`: `language` stays `HubWide` in `configreg`, its file on the board only.
- `PATTERN-config-strictness`: `languageconfig.Load` keeps `LoadOrTemplate`.
- `PATTERN-binary-change-reconcile`: an existing hub's `language.yaml` gains the new key only through `hubreconcile.Ensure`.
- `PATTERN-no-denied-recovery`: the reworded driver and rework sentences name no denied command; `goto` stays a verb the driver reports and never runs.
- `PATTERN-producer-pointer-rule`: the rework stencil points at the plan stencil's scope-addition route and never restates it.
- The plan and rework templates keep no `{{if}}`/`{{range}}` conditionals.

## Auto-mode assumptions

- D1's conflict route and its decline outcome go past the entry's wording-only Idea, as the meaning of "not on a par with the decision record"; no operator was asked.
- D5's prose list over a structured schema.
- The spec's verify model and the implementer's whole raw-run section are in scope, past the board's cited lines, because they carry the same Go text to the same readers.
- D7 leaves this hub's value to the operator.

## Open risks

- Between this task's deploy and the operator setting this hub's `verify_tiers`, plan writers here get the neutral rule and may omit the `go vet -tags tmux`/`-tags llm` compile steps, and Plan-Review checks no order.
- The list is read fresh at each spawn, so a mid-run edit of `language.yaml` can make a later Plan-Review round check a plan against rules it was not written to; reconciling that is the operator's.

<!-- lyx:carry-over Plan-Review -->
- Plan-Review, round 2: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/plan/round-2-review.md; fixer report: _lyx/reviews/plan/round-2-fixer-report.md.
  - card4-test-helper-resign-missing-wrappers (class scope, severity MEDIUM, open in rounds 2)
  - tier-skip-bound-unstated-in-intents (class scope, severity MEDIUM, open in rounds 2)
<!-- /lyx:carry-over Plan-Review -->

## Acceptance criteria

- `languageconfig.Load` resolves `verify_tiers`: an absent key and an absent file give the empty list, a list gives its entries in order, and each refused shape (non-list, non-string entry, empty entry, surrounding whitespace, line break) fails with the way forward; a local override holding the key is refused.
- An invalid `verify_tiers` refuses a real spawn path that loads `language.yaml` with the way forward.
- With a non-empty list, the rendered plan prompt, the Plan-Review rubric as `ReadRubric` fills it, and both implementer renders carry every entry as a bullet in order, verbatim, an entry holding `{{` included.
- With the empty list, the plan prompt, the rubric and both implementer renders each render `none` beside their fallback sentence and read correctly, the implementer section still limits raw runs to the listed packages and its guard sentence, and none of them holds an empty bullet.
- No rendered plan, Plan-Review or implementer prompt for a hub with an empty list names `go build`, `go vet`, `go test`, `-tags tmux`, `-tags llm` or `go -C`, except in sentences describing the Go-coded guards and gates the Out list keeps, the implementer's `tmux`/`llm` compile-only sentence among them.
- The rework prompt calls the findings the rejecting reviewer's, names the alignment agent and the parent's review as their source, routes a Decision-overturning finding through the parent, plans a finding that overturns no Decision directly as no late scope addition under the plan stencil's route, and stops the whole rework without writing `coverage.md` on a decline or when no parent is recorded; the rendered Plan-Review rubric calls a finding a reviewer's claim and flags a card carrying a Decision-overturning finding with no `Added after Discussion` entry; the plan and rework prompts say they never ask the operator and name their parent requests.
- The driver prompt names no hand commit and no operator push; the advisor prompt has no "for its format".
- The stencil and spec scans (`PATTERN-no-denied-recovery`, `PATTERN-markdown-link-integrity`) pass over the edited files.

## Notes for the plan writer

- The stencil fillers are `internal/seatengine` from `loomengine.PlanTable` and `loomengine.ReworkTable`, and `internal/websterengine`'s two implementer value maps; the rubric's is `shedadapters.ReadRubric`.
- `languageconfig.Load`'s callers are in the burler, loom and webster CLI wiring.
- The plan writer's lens forks get the Plan-Review rubric by path (`loomengine` twill wiring) and read it unfilled, as they already read its other markers; their context holds the writer's filled plan prompt, which carries the list.
- `internal/languageconfig` has a row in `cmd/lyx/testdata/test-budget.yaml`; a card adding top-level tests past it lists that file under `Edit:`.
- `ConfigTemplate` carries one-line `key: value # comment` entries; the new key's comment says what an entry holds and that the empty list is the neutral default.
