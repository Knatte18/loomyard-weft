# Support log — loom-stencil-wording

## Interview

Autonomous mode, no advisors: every turn below is the writer's own exploration and pick.

### Turn 1 — board and related entries

Read the task entry and the entries it touches: `webster-fork-stencil-fixes` (depends on this task; edits other webster and pattern stencils, no overlap with the sites here), `orch-role-stencil-fixes` (orch role stencil only), `dotnet-verify-protocol` and `dotnet-scoped-verify` (a .NET hub's verify protocol and toolchain declaration; this task only removes Go text from the prompts and adds a hub list the .NET tiers can fill).
Confirmed the cited lines on this branch: rework `:12`, `:14`, `:29`; rubric rework answer-key bullet and `Verify coverage` bullet; driver `:67`; advisor `:10`; plan `:15`, `:34-42`, `:201-205`; implementer raw-run section.

### Turn 2 — where the tiers come from

`internal/languageconfig` is the hub-wide `language.yaml`, one key `skills`, and its doc says lyx hard-codes no language.
`landing.yaml` already holds `publish_verify`.
The card gate, batch gate, `lyx gate test` and the shell guard's raw-go row are Go-coded and installed in every hub, so the sentences describing them stay; only the prescriptive tier text moves.
`loom-plan-spec.md`'s `## verify model` restates the Go order, and the plan writer and Plan-Review read it, so it joins the scope.
Picked a list key `verify_tiers` in `language.yaml` (auto-pick).

### Turn 3 — the rework findings

PR rejections come from the alignment agent's report or the orch's reviewer fork; operator rulings live in the decision record via `lyx loom decision add`.
"Not on a par with the decision record" needs an outcome for a finding that overturns a Decision; the rework writer already inherits the plan stencil's late scope-addition route, so it is reused (auto-pick).
PR-Rework's Go side only copies `coverage.md`; per-finding coverage is a stencil and Plan-Review rule, so no code change follows.

### Turn 4 — this hub's value

`language.yaml` is on the board, outside the worktree, so no card can set this hub's value; the PR description names the command and `languageconfig`'s doc carries the Go example (auto-pick, revised in Turn 5).

### Turn 5 — lens review

Six forks, one per lens; every fork returned a report.

- premises, MEDIUM, `ReadRubric`'s strict fill refuses an empty marker, so the rubric needs the key in its fixed map, rendered `none` when empty: fixed in D6, Notes.
- premises, LOW, Notes pointed at the wrong rubric filler: fixed, names `shedadapters.ReadRubric`.
- premises, LOW, the "already inherits the route" premise is ambiguous: fixed, D1 now has the rework stencil name the route for Decision-overturning findings.
- premises + scenarios + leanness, LOW/MEDIUM, the `--set` command breaks on the entries' quotes, and a PR description is the Describe session's, not a card's: fixed, D7 drops the PR description and points the operator at the editor.
- premises, LOW, the implementer body has two value maps: fixed in D6 and Acceptance.
- scenarios, MEDIUM, no empty-list behaviour for the rubric and implementer: fixed in D6 and Acceptance.
- scenarios, MEDIUM, a declined parent recording has no outcome: fixed, D1 stops and reports; Acceptance covers it.
- scenarios, MEDIUM, an invalid `verify_tiers` blocks every `language.yaml` consumer: fixed, stated as D5's bound and covered in Acceptance; a separate loader rejected (see Rejected alternatives).
- scenarios, LOW, the attack-surface bound of the new channel unstated: fixed in D5's bound.
- scenarios, LOW, a mid-run list change: fixed, Open risks.
- scenarios, LOW, template-like text in an entry: fixed, D6 says verbatim, Acceptance covers `{{`.
- goal-scope + board-fit, MEDIUM, no check against `dotnet-verify-protocol`: fixed, D5's rationale maps the .NET tiers.
- goal-scope + board-fit, MEDIUM/LOW, this hub's value has no owner and hides a deferral: fixed, D7 names the operator and why hub config is theirs; the window stays in Open risks.
- goal-scope, LOW, D1 goes past wording: fixed, marked in Auto-mode assumptions.
- goal-scope, LOW, the spec in scope: already disclosed, no change.
- goal-scope, LOW, the implementer heading names Go: fixed, D6 retitles it.
- goal-scope + board-fit, MEDIUM/LOW, the dependent `webster-fork-stencil-fixes` shares files: fixed, D6 states this task edits none of its cited ranges.
- board-fit, MEDIUM, the dependent's rename tool would land in `language.yaml` in another shape: rejected, that entry decides its own key; D5 establishes `language.yaml` as the module, and prescribing its shape here would design another task.
- board-fit, LOW, `verify_tiers` partly answers `dotnet-scoped-verify`: fixed, Scope Out says the list is prompt guidance only.
- board-fit, LOW, a post-landing step belongs in `ORCHESTRATOR.md`: rejected, that file is unbuilt and the step is the operator's, not the orch's.
- pattern-fit, LOW, config text outside the `PATTERN-no-denied-recovery` scan: fixed, named in D5's bound.
- pattern-fit, LOW, test budget: fixed, Notes.
- pattern-fit, LOW, `LocalKeys`: fixed, D5.
- pattern-fit, LOW, marker value shape: fixed, D6 says bullets only.
- leanness, MEDIUM, Scope In repeats the Decisions: fixed, pointers only.
- leanness, LOW, Scope Out reasons and the D7 repeat: fixed, the reason cut; the D7 line kept as a pointer.
- leanness, LOW, non-constraints: fixed, `PATTERN-tolerant-records`, line-break and docs lines cut.
- leanness, LOW, Auto-mode assumptions restate the ledger: fixed, shortened.
- leanness, LOW, generic gate acceptance bullet: fixed, scans only.
- leanness, LOW, Turn 1 as a checklist: rejected, the support log is where the premise check belongs.

## Rejected alternatives

- **A structured per-tier schema (`command`, `run|compile`, `reason`)** for `verify_tiers`: encodes Go's tier shape, and the stencils need prose rules (nested-module form, raw-run limits) a schema would not hold.
- **One prose string key**: a long single line in a one-line-per-key config, and no clean bullet rendering.
- **A hub-editable stencil fragment holding the tiers**: the seeded default would be some language's text, so a fresh hub still gets Go; config is where hub facts live.
- **A template default of this repo's Go rules**: every new hub would inherit Go, the defect itself.
- **Rendering `publish_verify`'s command into the prompts**: the list's rule names which tier Publish runs; a second rendered value adds nothing.
- **Generalising `verify-module-wide`, `verify-nested-module`, `lyx gate test` and the raw-go guard**: Go-coded gates are `dotnet-scoped-verify`'s toolchain question, not wording.
- **Letting the rework writer dispute a finding in `coverage.md` with no card**: contradicts the stencil's "never answer a finding with a coverage entry alone" and the round rule; the parent route covers the conflict case.
- **Fixing `lyx loom reject`'s help ("the operator's rejection")**: the operator may run the verb by hand, so it is not wrong.
- **Naming the `--set` command in the PR description**: the description is the Describe session's, no card writes it, and the entries' quotes break a single-quoted `--set` line.
- **A separate loader for `verify_tiers`** so a bad value cannot block `skills`: the keys share one file, whose parse error blocks both anyway, and one `Load` keeps the module's single read.
- **A PATTERN entry or scan test that hub-generic stencils name no Go command**: false while the guard sentences rightly name Go-coded gates; the acceptance criterion pins the rendered prompts instead.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick): where the verify tiers live. Picked `language.yaml`'s new `verify_tiers` list; see D5.
- Q2 (auto-pick): which sites render it. Picked the plan template, the Plan-Review rubric, the implementer body and the plan spec's verify model; see D6.
- Q3 (auto-pick): the empty-list fallback. Picked a neutral sentence in each stencil (build, then test every targeted package), never Go-embedded prose.
- Q4 (auto-pick): what a rework finding that overturns a Decision does. Picked the plan stencil's existing parent-recording route; see D1.
- Q5 (auto-pick): this hub's value. First picked the PR description naming a `--set` command; after the lens review, picked the doc example plus the operator setting it in the editor; see D7.
- Q8 (auto-pick): what a rework writer does when the parent declines to record a Decision-overturning finding. Picked stop and report; see D1.
- Q9 (auto-pick): the rubric's empty-list value under `ReadRubric`'s strict fill. Picked `none`, matching its notes; see D6.
- Q6 (auto-pick): whether the guard-describing Go sentences (`verify-module-wide`, `lyx gate test`, slot-busy exit) stay. Picked keep: the guards run in every hub.
- Q7 (auto-pick): whether `loom-plan-spec.md` is in scope though the entry names three stencils. Picked yes, its verify model sentences only.
