# Card 9 — loom-row-fixer-wiring

**Create:**
- `contracts/stencils/gatefix/gatefix-body-darn.md`
- `contracts/stencils/gatefix/gatefix-body-plan.md`
- `contracts/stencils/gatefix/gatefix-body-discussion.md`
- `contracts/stencils#GatefixBodyDarn`
- `contracts/stencils#GatefixBodyPlan`
- `contracts/stencils#GatefixBodyDiscussion`
- `internal/loomcli#pairSlugOf`
- `internal/loomcli#rowVisitDir`
- `internal/loomcli#gateFixerInputs`
- `internal/loomcli#gateFixerSeam`
- `internal/loomcli#fixerSpec`
- `internal/loomcli#TestRowVisitDir`
- `internal/loomcli#TestGateFixerSeam`
- `internal/loomcli#TestFixerSpec`
- `internal/loomcli#TestPairSlugOf`

**Edit:**
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#intentionallyNil`

**Extends:**
- `contracts/stencils#entries`
- `contracts/stencils#TestStencils_TopLevelMarkers`
- `internal/loomcli/doc.go`

**Uses:**
- `internal/loomcli#TestWire_EverySeamFilled`
- `internal/gatecli#originSlug`
- `internal/loomcli#sessionRegistrar`
- `internal/testkit/stencilkit#Seed`
- `internal/hubforge#`

**Intent:**
Wire `Env.GateFixer` for the Darn-Write, Plan-Write and Discussion-Write rows (D1, D2, D5, D8), with one body stencil per row caller.

- `pairSlugOf(location)` returns `location.WorktreeName` when `fabricengine.ReadOrigin(location)` finds an origin, and `""` on the prime or outside a hub, as `gatecli`'s `originSlug` reads it; a read error is returned.
- `rowVisitDir(scratchDir, statusPath, statusLockPath, row)` reads the run's status with `state.ReadJSON[shedengine.Status]` and returns `<scratchDir>/gatefix/<row>-<len(History)>-<newest entry's Repeats>` (D8).
  A call that ends appends a history entry or folds into the newest and a goto appends one, while an interrupted step writes none, so a crash re-step and a resume of a failed stop that appended no history entry keep the same directory, and a goto, a resume of a halt that appended one and a fresh row entry get a new, empty one.
  A missing status file is an error naming the path.
- `gateFixerInputs` carries the location's pair slug, the hub's runproc directory, the shuttle runner (starter and stopper), reed, the engine set, the shuttle config, the anchor root, the model registry, the gate config, the stencils dir, the language skills, the parent name, the friction dir, the run scratch dir, the status and status-lock paths, and the told paths each body names.
- `gateFixerSeam(in)` returns the `Env.GateFixer` seam.
  With an empty pair slug it returns the request's spec unchanged, so a row on the prime or outside a hub spawns no fixer and its gate failure reaches the writer as today (D1).
  Otherwise it resolves the request's model-spec through the registry into a `shuttleengine.NewModelChoice`, refusing an alias the registry does not hold with the setting's way forward, and returns `gatefixer.WrapRowGate(req.Spec, req.Entry, rf)` with `rf.Fixer` a `gatefixer.NewStrandFixer` whose `Register` is `gatefixer.Registrar(reed, runProcDir, slug, gatefixer.RoleGateFixer)`, `rf.VisitDir` over `rowVisitDir` for `req.Row`, `rf.Timeout` the gate config's `FixerTimeout()`, `rf.Forked` and `rf.SkipForked` from the request, and `rf.Spec` the closure `fixerSpec` returns.
- `fixerSpec(in, req, choice)` returns the closure that renders a fixer's prompt with `gatefixer.RenderPrompt` and builds its `gatefixer.Spec` under `RoleGateFixer`, with the hub's language skills, `choice` and the gate config's timeout; it is its own member so a test reads the spec the wrap would spawn without driving a strand.
  The body is chosen by the request's entry name from one table: `verify` takes `gatefix-body-darn` and `segmentcolor.Darn`; `plan` takes `gatefix-body-plan` and `segmentcolor.Plan`; `discussion` takes `gatefix-body-discussion` and `segmentcolor.Discussion`; an entry outside the table is refused naming it, a fail-loud wiring guard since card 8 sends only those three.
  Body values: Darn-Write's `fix_gate` is the gate config's `AgentGateCall("darn-write")`, beside `worktree_root` and `decision_record_path`; Plan-Write's `plan_dir`, `lens_review_path` and `recheck_command`, `lym loom validate-plan --lens-review` for a forked request and `lym loom validate-plan` otherwise; Discussion-Write's `decision_record_path`, `support_log_path`, `relevant_path` and `recheck_command` `lym loom validate-discussion`.
  The friction note is `gate-fixer-<row>` under the friction dir when Tier 2 is on, and the parent name is the run's.
- `loomCLI.wire` sets `Env.GateFixer` to `gateFixerSeam` over the values it already holds, and `intentionallyNil` drops `GateFixer`.
- The body stencils set each caller's edit scope, commit rule, re-check and leave-list (D5), each with no `{{if}}` or `{{range}}`, and every fixer's shared rules: read the report and the source behind each failure, fix every failure it can in one pass, never delete, skip or weaken a test, re-run the failed check first and then the whole check, write the reply as the final action, and leave, naming it in the reply, any failure whose fix would add, remove or change a decision, an acceptance criterion, a PATTERN change, a card or a card's scope, or a test's intended assertion, or needs a choice the record, plan or card does not make.
  - `gatefix-body-darn`: source only, never `_lym`; each fix committed `fix: <summary>`; the worktree left clean; re-check with `{{.fix_gate}}`.
  - `gatefix-body-plan`: only the files under `{{.plan_dir}}` but the lens review record `{{.lens_review_path}}`, which is the writer's and always left, as is a missing or incomplete lens review; no commit, since the row commits them; re-check with `{{.recheck_command}}`.
  - `gatefix-body-discussion`: only the three discussion files; never an `### Added after Discussion` entry, a suffixed `## PATTERN changes` bullet or a carry-over block; no commit; re-check with `{{.recheck_command}}`.
  They are embedded and registered as card 2 registers the frame, with their marker rows in `TestStencils_TopLevelMarkers`.

Tests, tier0:
- `TestRowVisitDir` rows over a written status file: an unchanged history gives the same directory on a second read (crash re-step, failed stop); an appended history entry gives a new one (goto, halt resume, fresh entry); a fold into the newest entry gives a new one; a missing status file is an error naming the path.
- `TestGateFixerSeam` over a scripted gate config and model registry: an empty slug returns the spec unchanged; with a slug the fixer entry comes back with `MayHold` and `Final` set; a model-spec whose alias the registry does not hold is refused naming the setting's way forward.
- `TestFixerSpec` over a stencils dir from `stencilkit.Seed`: for each of the three entries the spec carries `RoleGateFixer`, the gate config's timeout, the model choice and its row's segment; the darn prompt carries the `darn-write` gate call, the plan prompt carries `lym loom validate-plan` unforked and `lym loom validate-plan --lens-review` forked and the lens review path, and the discussion prompt carries `lym loom validate-discussion`; an entry outside the table is refused.
  The rows assert the values `fixerSpec` fills, never a body's static prose, which the Plan-Review rubric's test economy rule keeps out of tests.
- `TestPairSlugOf`, tier1, over a `hubforge` hub: a pair worktree answers its slug and the prime answers `""`.
- `TestWire_EverySeamFilled` holds `GateFixer` filled once `intentionallyNil` drops it.

**ImpactSummary:** `loomCLI.wire` fills one more seam; the three gated rows of tapestry and darn runs on a pair now wrap their gate, and every other row and every prime or standalone run is unchanged.
