## Goal

Each launch writes its manifest, a parity test compares the engines per seat, and lym shuttle manifest prints both

Takes: prompt-engine-per-launch's per-producer render functions taking the engine; codex-standing-instructions' injectable hook runner and sandbox probe; every Codex delivery of codex-standing-instructions and codex-question-deny-nudge, which the parity test then finds present; codex-wording-engine-blocks' engine blocks, which the test renders per engine
Delivers: the launch manifest read back from each engine's Prepare, written as a record into the run directory and as a copy into the run-directory root's manifests directory, skipped by the orphan sweep and the run lookup by strand; the parity test with its producer table and the scan for unlisted launch-spec literals; the lym shuttle manifest verb; the bench rule on codex-engine-trial, and on codex-writer-bench, the draft that replaced codex-writer-bench-osl on 2026-11-16 and already runs the real producer, so D11's osl section lands there as a check that its body states the rule whole
Exports: the manifest record and the lym shuttle manifest verb, which the Codex benches read

## Scope

The part changes these packages:

- internal/shuttleengine (about 300 changed lines)
- internal/shuttleengine/claudeengine (about 200 changed lines)
- internal/shuttleengine/codexengine (about 200 changed lines)
- internal/shuttlecli (about 200 changed lines)
- internal/shuttleparity (about 600 changed lines)

## Decisions

### D1 — A launch manifest read back from each launch, compared per seat

Each engine reads a launch manifest back from the launch its `Prepare` produced: the launch command, the files it wrote and the settings it passed.
The manifest holds one item per thing the session receives, keyed by channel and an engine-neutral key.
The compared channels are:
prompt engine, edit directive, standing passages by heddle slug, instruction sources by origin, skill loads, skill listing, plugin session-start contexts by plugin and hook, guard rows and spec-level denies, the batching nudge, and notices (deny notice, creel trust sentence, subagent passage).
A Claude item that Claude Code loads itself, such as the user's global instruction file, is listed from `claudeengine`'s resolution of it as a native item.
An item from a shared heddle target (`agents-file`, `agents-prompt`) is keyed by that shared target and compared,
though it reaches each engine through that engine's own file or prompt target.
Only an entry stored under a target aimed at one engine (`claude-file`, `claude-prompt`, `codex-file`, `codex-prompt`) is keyed to that engine and not compared,
since that target is a repository's choice for one engine.
Engine settings (Claude Code's settings document, Codex's sandbox flags) are listed and not compared.
`Prepare` writes the manifest into the run directory as a record, marking each plugin session-start context delivered or omitted;
an omitted item counts as missing in the record's verdict.
A launch's `Prepare` also writes a copy named by the run id into a `manifests` directory at the run-directory root,
which a done run's cleanup leaves in place;
the sweep of orphaned run directories and the run lookup by strand skip that directory, since it holds no `run.json`.
Bound: nothing removes those copies.

The parity test builds one launch spec per engine for every seat in `seatengine`'s tables and for every other producer that builds a launch spec:
the same role and skills, and the producer's prompt rendered for that engine.
A producer outside `seatengine` takes part through the test's producer table, which holds one entry per function holding a production launch-spec literal,
and a scan fails on such a literal whose enclosing function has no entry in the table.
A launch-spec literal is a `shuttleengine.Spec` composite literal that sets `Prompt`: the spec a producer passes to a start, a run or an attach-then-run call.
A literal that sets no `Prompt` launches nothing and is skipped by this scan, by D4's scan and by the producer table;
such are the attach probes of `awaitLiveJudge` and `awaitLiveSeed` in `internal/shedadapters/bouncer.go`, which carry only `OutputFiles`, `Role`, `Segment` and `Round`.
A producer that fills its prompt inline in the function that launches it, as that file's `judgeCall` and `runSeedSpawn` do,
gets its prompt render split out of that function in its own package, so its table entry has a renderer to call with the engine.
The scan skips the orch places D2 skips, the shuttle CLI, whose `lym shuttle run` spec is hand-written (D4) and whose `lym shuttle open` spec carries no role,
and the test-support packages under `internal/testkit`.
It runs both engines' `Prepare` on fixtures with a faked sandbox probe and hook runner, and fails when the Codex manifest lacks a compared item of the Claude manifest.

Rationale: a manifest read back from the launch drops an item when a delivery is dropped, and comparing keys rather than text lets each engine word its copy its own way.

### D10 — A verb prints each engine's manifest for a seat

`lym shuttle manifest` takes a seat and prints the manifest each registered engine gives it in the current worktree, with the seat's prompt rendered for each engine, and the parity verdict.
It exits non-zero and names each missing item when the Codex manifest lacks one.
It runs each engine's `Prepare` into a temporary run directory it removes before it exits,
with a sandbox probe that reports the configured mode and a hook runner that lists each plugin hook unrun, both production code the parity test shares.
Its verdict compares plugin contexts by plugin and hook key alone, so an unrun context counts as present.
Bound: it starts no session and runs no plugin hook or sandbox probe;
it writes nothing outside that temporary directory but the Codex home a Codex `Prepare` readies: the home's links, its trust record and its `config.toml`.
Its `Prepare` runs keep no lasting manifest copy, which D1 gives a launch only.
It lists plugin contexts by plugin and hook, and only a run's record says whether one was delivered.
Its size check of D5's `developer_instructions` argument leaves out the plugin session-start contexts,
so a seat it passes can still be refused at a real launch.

Rationale: a bench report needs the engine that did not run beside the one that did, without hand-assembling it, and the verb shares the test's code.

### D11 — The bench rule goes on the board

The project entry `codex-engine-trial` gains, under its Rules, the bench rule:
a Codex bench runs the real producer with only the model swapped through `run_settings` and never hand-builds Codex's prompt.
Its report opens with the run's own manifest record, from the copy D1 keeps in the run-directory root's `manifests` directory, and, for the engine that did not run, the output of `lym shuttle manifest` for the same seat.
The entry `codex-writer-bench-osl` gains a section saying its rerun follows that rule, so its hand-built prompt, scratch replay and single-session bounds no longer hold, and its replay route is redesigned before it reruns.

Rationale: the operator's rule binds every later bench, and how a real producer replays a landed task is that bench's design, not this task's.

### Added after Discussion (arbiter, 2026-11-16): The assist-trial schedule risk has no subject

Decision: Drop the open risk on codex-assist-review-trial and close Q14 as answered: this task takes no scheduling call on that trial. The three runs the trial chose (operator, 2026-11-15), prime-is-root-beam, hub-stop-button and machine-restart-shortcut, no longer exist as tasks: the operator merged them into adopt-build-resplit (2026-11-16) to be split again from scratch, and that entry carries no code_assist_reviewer_model setting, so no run with a Codex assist seat is queued or running. Any runs the trial picks afresh are the trial note's own choice, so this record states no scheduling bound.

Rationale: Q14 asked whether the three chosen runs go before or after this task lands. Board facts: lym board find code_assist_reviewer_model matches only the trial note, and adopt-build-resplit's body says the beam's remaining members were merged into it. So there is nothing to schedule, and choosing future trial runs belongs to codex-assist-review-trial, not to this task's design.

### Added after Discussion (arbiter, 2026-11-16): Round-3 review findings are answered by the record as it stands

Decision: Treat the round-3 findings d12-non-exec-calls-outside-cell-model (F1), d1-scan-reaches-probe-literals (F2) and d12-read-again-path-resolution-unstated (F3) as answered by D1, D4 and D12 as written; the plan carries D12's function_call rules, D1's launch-spec literal definition (a shuttleengine.Spec literal that sets Prompt, attach probes skipped, inline prompts split into a renderer) and D12's per-call cwd resolution as stated, with no further change.

Rationale: The Discussion review reached its hard cap at round 3 with these keys open. D12 now scopes the payload and cell sentences to nested calls, gives a non-Bash, non-apply_patch payload no nudge and leaves the rollout unread for it, and ends a lookup_run or edit_run chain at a function_call; the 2026-11-15 rollout holds 21 custom_tool_call exec items and four collaboration function_call items (two spawn_agent, two send_message) as D12 states. D1 and D4 skip a Spec literal that sets no Prompt; bouncer.go's awaitLiveJudge and awaitLiveSeed literals set only OutputFiles, Role, Segment and Round, while judgeCall and runSeedSpawn set Prompt inline. D12 resolves both read paths against each call's CommandExecution cwd; the 2026-11-16 spike rollout carries that cwd as a file:// URL and parsed_cmd paths as written (notes.txt).

### Added after Discussion (parent, 2026-11-17): D11's bench rule goes on the codexengine project

Decision: D11's bench rule, named for the project codex-engine-trial, is written under the ## Rules of project P1 codexengine, which codex-engine-trial was merged into on 2026-11-16.

Rationale: codex-engine-trial no longer exists as a board entry; codexengine holds its members and its body records the merge, so the rule reaches the same benches there.

### Added after Discussion (parent, 2026-11-17): D11 is done on the board by the orch; card 10 drops

Decision: The plan drops card 10. The orch wrote D11's bench rule on 2026-11-17: under the ## Rules of project codexengine, and as a closing line of codex-writer-bench's body, which already runs the real producer with only the model swapped. No card of this task writes the board.

Rationale: A plan card targets files in the worktree, and a board body is no such target; the board edit is a few lines that the orch makes directly, so neither a validator exemption nor a reshaped card is needed.

### Added after Discussion (parent, 2026-11-17): Every seat is compared, including those Codex cannot start

Decision: The parity tests compare every seat and producer, including fanned burler review and fix, Plan-Write and Discussion seats, Merriam and the loom driver. Prepare does not refuse an unmet requirement, only Start does, so the manifest exists for each; the Unmet skip in TestSeats and TestProducers is dropped.

Rationale: D1's acceptance criterion names every seat in seatengine's tables and every other producer, and its auto-mode assumption records no exemption table. A seat Codex cannot start today is the one most likely to drift, so it stays covered.

### Added after Discussion (parent, 2026-11-17): The webster fork deny is compared only on an engine that meets the fork requirement

Decision: The guard item webster-fork-deny belongs to fork support: it is compared only when the engine meets RequirementForks. Codex does not meet it today (its Start refuses a ForkSubagents spec, and its hook payload marks no subagent call), so on Codex the item is listed and not compared, and every other item of those seats is still compared. A test pins the tripwire: once Codex reports fork support, the item becomes compared and the parity test fails until Codex delivers a fork deny. No card builds a Codex fork deny.

Rationale: A Codex fork deny cannot be built now: Codex runs no fork subagent and its hook input carries no subagent marker, so the deny would guard nothing. Keying the item to claude would hide the gap for good; gating it on the fork requirement keeps the rest of every seat compared and makes the gap fail loudly the day Codex can fork.

### Added after Discussion (arbiter, 2026-11-17): Plan review round 3 closes at the hard cap with every finding fixed in place

Decision: The plan review ends unconverged on round 3 and the plan stands as the round-3 fixer left it. Card 6 (06-seat-parity) Seat.Specs gives each engine a spec whose ModelChoice is ModelChoice{Engine: engine} alone, with Model, Effort and Version cleared, and TestCompare's seatsOf row pins it; card 6 creates TestMain in internal/shuttleparity's testmain_test.go; cards 2, 5 and 6 (02-manifest-record, 05-engines-with-seams, 06-seat-parity) add interim testOnlyCodeAllowed entries that card 8 (08-shuttle-manifest-verb) deletes; card 5's build glyph carries its re-sign arrow; card 6 declares the reference and compared engine names as exported constants of internal/shuttleparity. Implement the cards as written.

Rationale: The one design-wrong finding, seat-specs-keeps-claude-model-params, is answered by the fix in place: codexengine.validateChoice refuses only a non-empty Version, an unlisted non-empty Effort or a non-bypass permission mode; claudeengine's validateEffort accepts an empty effort and resolveModelID returns the empty model when no version is given. Model and Effort reach only the launch command settings (buildLaunchCmd, Codex's model and effort args), which D1 lists and does not compare, so clearing them changes no compared item. The other four open keys are mechanical and the plan carries each fix as the review worded it.

### Added after Discussion (parent, 2026-11-17): The parity check names its two engines as constants

Decision: ReferenceEngine (claude) and ComparedEngine (codex) stay as named constants in internal/shuttleparity. The commit that adds them carries a Pattern-Departure trailer naming PATTERN-code-conventions, with the reason that the parity comparison is between two named engines by definition.

Rationale: The check exists to hold Codex to what Claude delivers; deriving the pair from the engine set or config would add a setting with one meaningful value. The departure is confined to the parity package and its CLI verb, and no other caller branches on an engine name.

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

<!-- lym:carry-over Plan-Review -->
- Plan-Review, round 3: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/plan/round-3-review.md; fixer report: _lym/reviews/plan/round-3-fixer-report.md.
  - interim-reachability-not-admitted-in-allowlist (class design, severity MEDIUM, open in rounds 3)
  - seat-specs-keeps-claude-model-params (class design, severity MEDIUM, open in rounds 3)
  - shuttleparity-package-lacks-testmain (class scope, severity BLOCKING, open in rounds 3)
<!-- /lym:carry-over Plan-Review -->

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - claude-local-instruction-name-redeclared (class consistency, severity BLOCKING, open in rounds 1)
  - codex-drops-omitted-hook-without-plugin (class design, severity MEDIUM, open in rounds 1)
  - heddle-file-target-engine-literals (class consistency, severity BLOCKING, open in rounds 1)
  - manifest-fields-unrun-and-verdict-untested (class design, severity MEDIUM, open in rounds 1)
  - manifest-report-of-untested (class design, severity MEDIUM, open in rounds 1)
  - parity-engine-names-hardcoded (class decision, severity MEDIUM, open in rounds 1)
  - production-passage-reader-untested (class design, severity MEDIUM, open in rounds 1)
  - report-tempdir-removal-untested-on-errors (class design, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria

- For every seat in `seatengine`'s tables and every other producer of a launch spec, the Codex manifest holds every compared item of the Claude manifest, and dropping any one Codex delivery from `Prepare` fails the test;
  a production launch-spec literal whose enclosing function has no entry in the producer table fails the scan, and a spec literal that sets no `Prompt` is skipped (D1, D5, D7, D8, D12).
- An item from a shared heddle target (`agents-file`, `agents-prompt`) is compared, so dropping its Codex delivery fails the test;
  an entry stored under a single-engine target is not compared, and a plugin context omitted at run time makes the run record's verdict fail (D1).
- `Prepare` writes the manifest record into the run directory for both engines and its copy into the `manifests` directory, the copy remains after a done run's cleanup,
  and a record with an unknown key still reads (D1).
- `lym shuttle manifest` prints both engines' manifests for a seat, exits non-zero naming a missing item, passes with plugin contexts listed unrun,
  and writes nothing outside a temporary run directory it removes and the Codex home, no lasting manifest copy included (D10).
- Review: the `codex-engine-trial` and `codex-writer-bench-osl` entries state the bench rule (D11).

## PATTERN changes

- `PATTERN-engine-parity` — adds: Launching a role on an engine: the session gets every skill, standing passage, instruction file, guard, hook and directive a session of that role gets on another engine, through its own channel; each engine's launch manifest is compared by test. (test)
