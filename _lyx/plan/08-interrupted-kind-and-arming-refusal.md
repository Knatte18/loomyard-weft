# Card 8 — interrupted-kind-and-arming-refusal

**Create:**
- `plan:internal/shedverbs#KindInterrupted` -> `const KindInterrupted = "interrupted"`
- `plan:internal/shedcli#kindlessRefusal` -> `type kindlessRefusal struct`
- `plan:internal/shedcli#reportArmError` -> `func reportArmError(out io.Writer, verb string, err error) int`
- `plan:internal/loomcli#hubConfigWayForward` -> `const hubConfigWayForward = "way forward: run lyx config reconcile --apply in the hub, then step again"`
- `pattern/PATTERN-batcher-registry.md`
- `pattern/PATTERN-shuttle-stop.md`
- `pattern/PATTERN-parent-directive.md`

**Edit:**
- `internal/shedverbs#StepKinds`
- `internal/shedverbs/doc.go`
- `internal/shedcli#shedCLI.resolvePersistentPreRun`
- `internal/shedcli#armFromSeed`
- `internal/shedcli#recipes`
- `internal/shedcli/cli_test.go`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli/wiring_test.go`
- `contracts/specs/refusal-spec.md`
- `PATTERN.md`
- `pattern/PATTERN-shed-verb-set.md`
- `pattern/PATTERN-review-round.md`
- `pattern/PATTERN-agent-name.md`
- `pattern/PATTERN-gogit-read-helper.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Rename:**
- `internal/shedverbs#TestStepKinds_IsExactlyFive` -> `plan:internal/shedverbs#TestStepKinds_IsExactlySix`

**Uses:**
- `internal/shedverbs#KindBootstrap`
- `internal/shedrun#MissingSeedMessage`

**Intent:** The kind vocabulary grows to six and a pre-step arming refusal carries a kind and a trace, D4's vocabulary change and D6.
`KindInterrupted` joins the const block as the kind of a child step that ended without an envelope; `StepKinds` lists six, the const block's comment and the renamed test say six, and the test's `want` gains the new constant.
Nothing in this card emits the kind yet; card 10's loop does.
In `internal/shedcli`, `armFromSeed` wraps its two kind-less refusals, the missing seed and the unsupported verb, in `kindlessRefusal`, whose `Error` and `Unwrap` pass the inner error through; `resolvePersistentPreRun` reports every arming error through `reportArmError`.
`reportArmError` prints a bare `output.Err` envelope for a `kindlessRefusal` or for any verb but `step`, and for `step` it logs the refusal with `logger.Warn`, then prints an `output.ErrFields` envelope carrying `kind: bootstrap` and `trace_file: logger.TraceFile()`, so the arming refusal reaches the driver as a `bootstrap` error with a trace holding it instead of a kind-less line.
The missing-seed refusal stays kind-less, as the existing test pins, and a missing seed is never re-seeded by a driver.
`recipes`' comment says six.
`cli_test.go` gains `TestReportArmError`, a table over: a kind-less refusal on `step` prints no kind; a plain arming error on `step` prints `kind: bootstrap` and a `trace_file`; the same error on `status` prints no kind.
In `internal/loomcli`, `wire` wraps a failure of the hub-wide landing config load with `hubConfigWayForward`, so the missing-hub-config refusal names its way forward in the message; the driver's disposition for `bootstrap` is the repair path, whose repairs are a closed list of `lyx` verbs, and a config fix lies outside it, so the driver escalates with the way forward.
`wiring_test.go` gains a row in the default-config wire test's package where the landing config is absent from the hub directory and asserts the error ends in the way forward.
`refusal-spec.md`'s shed section gains the rows `step arming refused` (correctness halt, kind `bootstrap`, way forward as the wrapped error names it) and `hub config missing` (correctness halt, `lyx config reconcile --apply` in the hub, then the step again).
`PATTERN-shed-verb-set.md` states the six kinds and names `KindInterrupted` as the one the loop emits.
`PATTERN.md`'s shed-verb-set line becomes "Adding a run, step, status, pause or goto verb: the generic bodies live in `internal/shedverbs` only, which derives no path and closes the step kinds at six. (test)" with its background link, and the overview is brought under the cap of the Shared Decision `pattern-overview-cap` by shortening six other lines, each detail moving into the entry's background file, no invariant dropped:
`PATTERN-review-round` to "Running a review and fix round: review on disk before any target is touched, every finding fixed or disputed with evidence, converged only on a judge verdict." (the round-2 fresh-key rule and the `## Open risks` carry go to its background);
`PATTERN-batcher-registry` to "Choosing webster's execution unit: the batch the active `batcher.yaml` profile's batchifier derives through `internal/batcher`'s registry; the partition is recorded in `state.json` at first init." with a new background file carrying the `active:` key and the rebaseline rule;
`PATTERN-agent-name` to "Forming or parsing an agent name: only `internal/agentname`, formed once by reed, and the spawning module owns its role names as constants." (the two name grammars go to its background);
`PATTERN-shuttle-stop` to "Stopping a shuttle run's strand from Go outside `internal/shuttleengine`: only through shuttle's stop verb, which settles the run's record first; the driver and orch strand removals are exempt." with a new background file naming `loomcli` and `orchcli` and why no attach probe or Wait reads their records;
`PATTERN-parent-directive` to "Writing a spawned role's top-level stencil: it renders the parent directive, and no stencil tells an agent to ask the operator; the orch stencils are exempt. (test)" with a new background file carrying the `{{.mode_rules}}` clause;
`PATTERN-gogit-read-helper` to "Reading through go-git in `internal/gitrepo`: only through `readGoGit`, which retries a whole read once after a repack. (test)" (the sole-caller-of-`goGit()` clause goes to its background).
`test-budget.yaml` raises `internal/shedcli` from 17 to 18.

**ImpactSummary:** `StepKinds` gains an entry no code branches on yet; the shedcli pre-run keeps every kind-less refusal kind-less; `wire` only wraps one error; the PATTERN edits reword six overview lines and move their detail to background files.
