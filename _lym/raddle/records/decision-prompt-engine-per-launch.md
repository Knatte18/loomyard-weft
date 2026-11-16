## Goal

Every producer renders its prompt and edit directive for the engine its launch resolves to

Takes: nothing from another part; it waits for the board task weaver-implementer, which also changes internal/websterengine, so the two never run side by side
Delivers: a launch spec that carries its prompt engine, or the hand-written mark from lym shuttle run alone; a shuttleengine call that returns the engine name a model choice resolves to; Prepare refusing a mismatched or unset prompt engine; the orch spec pinned to Claude; one full edit directive per engine block with the shared stencil removed; the D4 scan over production launch-spec literals
Exports: for every producer outside seatengine, one prompt render function that takes the engine name and needs no session, split out of its launching function where that function fills the prompt inline (bouncer.go judgeCall and runSeedSpawn, websterengine, treadleengine, samplerengine, landingshed, frictionengine, mergeresolve); launch-manifest-parity's producer table calls these, and editdirective.Directive takes the engine

## Scope

The part changes these packages:

- internal/shuttleengine (about 120 changed lines)
- internal/shuttleengine/claudeengine (about 40 changed lines)
- internal/shuttleengine/codexengine (about 40 changed lines)
- internal/shuttleengines (about 150 changed lines)
- internal/shuttlecli (about 40 changed lines)
- internal/orchcli (about 20 changed lines)
- internal/editdirective (about 40 changed lines)
- contracts/stencils (about 250 changed lines)
- internal/seatengine (about 40 changed lines)
- internal/sheddriver (about 30 changed lines)
- internal/treadleengine (about 60 changed lines)
- internal/samplerengine (about 40 changed lines)
- internal/websterengine (about 120 changed lines)
- internal/landingshed (about 40 changed lines)
- internal/gatefixer (about 30 changed lines)
- internal/loomcli (about 30 changed lines)
- internal/dobbyengine (about 30 changed lines)
- internal/shedarbiter (about 30 changed lines)
- internal/frictionengine (about 40 changed lines)
- internal/mergeresolve (about 40 changed lines)
- internal/shedadapters (about 120 changed lines)

## Decisions

### D3 — One edit directive per engine, rendered for the launch's engine

The edit directive has one copy per engine, in its engine block, and each copy carries every rule the shared directive `contracts/stencils/edit/edit-directive.md` states today:
Go formatting through `lym loom fmt`, the scratch-file rules, renames, and work in another worktree only when the parent or operator says so.
The shared directive's text becomes the Claude copy, the shared stencil is removed, and the thinner assist copies are replaced, so an assist seat gets the full directive too.
`editdirective.Directive` takes the engine, every producer passes the engine its launch resolves to, and the edit-rule guard exempts the engine copies in place of the shared stencil.
Bound: a hub that edited its seeded copy of the shared directive loses that edit, since nothing reads the shared stencil any more;
a hub that edited its seeded engine copy keeps that copy as it is, without the rules the shared directive adds, since `stencilstore` never overwrites an edited stencil.

Rationale: `editdirective.EngineDirective` reaches only the assist seats today, so every other producer renders the Claude-worded directive on any engine, and both engine copies lack rules the shared one has.

### D4 — A prompt rendered for one engine never launches on another

Every launch spec carries the engine its prompt was rendered for, set by the producer from the same model choice it launches with.
A model choice that names no engine resolves to the engine set's default,
so a new `shuttleengine` call returns the engine name a model choice resolves to, the set's default name for an empty choice;
every producer passes that name to its prompt renderer and to the spec.
`Prepare` refuses a spec whose prompt engine differs from its own, or is unset, naming both engines and the producer's model setting as the way forward.
Bound: only `lym shuttle run`, from inline `--prompt` text or a `--prompt-file`, marks its spec hand-written in place of a prompt engine;
that spec is not refused, and its manifest names the prompt hand-written.
A scan fails on a production launch-spec literal (D1) that sets neither, and on a hand-written mark set outside `internal/shuttlecli`;
a spec literal that sets no `Prompt`, such as an attach probe's, is skipped.
The orch spec names the Claude engine and its prompt engine explicitly, so the shuttle default engine never gives Codex the orch role.

Rationale: D2 and D3 only help when the producer renders for the engine that runs the seat; this turns a missed producer into a refusal.

### Added after Discussion (arbiter, 2026-11-16): The assist-trial schedule risk has no subject

Decision: Drop the open risk on codex-assist-review-trial and close Q14 as answered: this task takes no scheduling call on that trial. The three runs the trial chose (operator, 2026-11-15), prime-is-root-beam, hub-stop-button and machine-restart-shortcut, no longer exist as tasks: the operator merged them into adopt-build-resplit (2026-11-16) to be split again from scratch, and that entry carries no code_assist_reviewer_model setting, so no run with a Codex assist seat is queued or running. Any runs the trial picks afresh are the trial note's own choice, so this record states no scheduling bound.

Rationale: Q14 asked whether the three chosen runs go before or after this task lands. Board facts: lym board find code_assist_reviewer_model matches only the trial note, and adopt-build-resplit's body says the beam's remaining members were merged into it. So there is nothing to schedule, and choosing future trial runs belongs to codex-assist-review-trial, not to this task's design.

### Added after Discussion (arbiter, 2026-11-16): Round-3 review findings are answered by the record as it stands

Decision: Treat the round-3 findings d12-non-exec-calls-outside-cell-model (F1), d1-scan-reaches-probe-literals (F2) and d12-read-again-path-resolution-unstated (F3) as answered by D1, D4 and D12 as written; the plan carries D12's function_call rules, D1's launch-spec literal definition (a shuttleengine.Spec literal that sets Prompt, attach probes skipped, inline prompts split into a renderer) and D12's per-call cwd resolution as stated, with no further change.

Rationale: The Discussion review reached its hard cap at round 3 with these keys open. D12 now scopes the payload and cell sentences to nested calls, gives a non-Bash, non-apply_patch payload no nudge and leaves the rollout unread for it, and ends a lookup_run or edit_run chain at a function_call; the 2026-11-15 rollout holds 21 custom_tool_call exec items and four collaboration function_call items (two spawn_agent, two send_message) as D12 states. D1 and D4 skip a Spec literal that sets no Prompt; bouncer.go's awaitLiveJudge and awaitLiveSeed literals set only OutputFiles, Role, Segment and Round, while judgeCall and runSeedSpawn set Prompt inline. D12 resolves both read paths against each call's CommandExecution cwd; the 2026-11-16 spike rollout carries that cwd as a file:// URL and parsed_cmd paths as written (notes.txt).

## Constraints

- `PATTERN-shuttle-provider-seam`: Claude's user-setup resolution, session scope and file locations stay in `claudeengine`, and reach `codexengine` only through `shuttleengine` interfaces wired in `internal/shuttleengines`; the rollout reading for the batching nudge stays in `codexengine`, and the read-only lookup check both engines' nudges use sits in its own package under `internal/shuttleengine`.
- `PATTERN-stencil-ownership`: engine text in engine blocks, resolved by `internal/stencilstore`'s one resolver; skills stay typed on the launch spec, and the skill listing is no request to load one.
  Its background's edit-directive section changes with D3: the engine copies are the directive's only statement.
- `PATTERN-heddle-instructions`: the shipped-entry changes and the new target live in `internal/heddleengine`; `HEDDLE.md` states the new target, and the background names the hub reconcile as a file-target render site.
- `PATTERN-child-processes`: the plugin hook runs are logged, waited on and registered as the pattern says.
- `PATTERN-tolerant-records`: the manifest record is written through `internal/state` and read tolerantly.
- `PATTERN-config-layers`: the new codex.yaml keys ship through `configengine` with their template defaults.
- `PATTERN-cli-cobra`: the verb joins shuttle's cobra subtree.
- `PATTERN-test-speed`: the parity test and the scans run at `tier0` on fixtures, and no test pins a stencil's prose.

## Auto-mode assumptions

- D1: no exemption table; every compared difference found has a Codex channel.
- D6: subagents run on the session's model; 16 sits above the widest fan any stencil asks for.
- D12: the nudge keeps separate settings per engine rather than moving to a shared module, which would need a config reinterpretation.

## Open risks

## Acceptance criteria

- Every producer's prompt for a Codex launch holds the Codex edit directive, an assist seat gets the full directive, and no stencil named `edit-directive` remains outside the engine blocks (D3).
- Review: each engine's edit directive carries every rule of the shared directive it replaces (D3).
- A producer whose model choice names no engine sets the default engine's name as its prompt engine and renders for it;
  `Prepare` refuses a spec whose prompt engine differs from its own or is unset, naming the way forward; a hand-written `lym shuttle run` spec, from inline text or a file, launches with its manifest saying so; a production launch-spec literal setting neither, and a hand-written mark outside `internal/shuttlecli`, fail the scan; the orch launches on Claude under a Codex default engine (D4).
