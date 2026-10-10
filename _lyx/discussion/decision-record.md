# Decision record: quick-recipe

## Goal

Add the `darn` task recipe: a batten-driven run for small changes with one LLM producer that reads the board entry, edits, tests and commits, then the usual Publish, PR-Gate and Finalize.
A failed verify re-prompts that same producer up to a configured budget, and a spent budget halts the run for the parent.
Land the four fixes the darn run depends on in the shared Publish, Finalize, teardown and reflection code: a verify log per attempt (#502), a Finalize that is not blocked by the parent's own batten records (#501), a teardown that settles the run's board claim, and a halt note that attributes a dirty tree to the right worktree.

## Scope

In:

- A second embedded recipe, `contracts/recipes/darn-recipe.yaml`, built by the loom recipe builder from the same registry, with its row names pinned the way loom's are.
- The `Darn` producer: one `SingleLLM`-backed row with a new stencil, a `verify` gate and the existing `description` gate, re-spawned with the operator's findings after a PR-Gate rejection.
- Recipe selection: the board entry's existing `recipe` field, accepted by batten's child seeding and read by loom's bootstrap, with the `darn` name added to the closed seed vocabulary.
- A hub-wide `darn.yaml` config module: writer model, writer timeout, the verify command and the verify attempt budget.
- A told verify-command source for the landing producers and the verify gate, so loom tells the plan's `## verify:` and darn tells `darn.yaml`'s.
- Per-attempt verify logs in `internal/verifytree`.
- Finalize committing the parent weft's run records before its parent-side merge.
- Pair teardown settling the board claim: marking a landed run's entry done and clearing an abandoned run's.
- The Go halt note carrying the parent pair's fabric status when a halt names a dirty worktree, and the reflection stencil reading it.
- The orch role stencil and the board skill and CLI help naming the darn recipe.
- Docs in the same commit: `docs/overview.md`, the package docs, `contracts/specs/refusal-spec.md`, `contracts/specs/shed-recipe-spec.md`, and the PATTERN entries this record names.

Out:

- A size guard or any automatic choice between loom and darn; the operator picks the recipe at task creation.
- A `lyx darn` command tree or any new CLI verb; a darn run is driven, approved, rejected, resumed and moved with the existing `lyx loom` and `lyx batten` verbs.
- A `Describe` row, Discussion, Plan, Webster, Bouncer or Burler rows in darn.
- Per-task darn configuration in a pair's `_lyx/config/`.
- Changes to the driver stencil, which stays recipe-blind.
- The Windows verify process-tree kill (`verify-watchdog`), `lyx cleanup`, `master-self-fix` and the convergence log, which stay their own entries.
- Migration of any in-flight run.

## Decisions

### D1. `darn` is a recipe driven by the loom module, not a new CLI module

Decision: `darn` joins `loom` and `batten` in `shedrun.RecipeNames`, `internal/shedcli`'s table maps it to `loomcli.ArmAt` with loom's bootstrap verb, and `lyx loom start` builds the darn shed when the worktree's seed records `darn`.
`lyx loom start` never writes a recipe over an existing seed: with no seed it seeds `loom`, with a seed it keeps the recorded recipe.
Loom's arm, the one wiring `loomcli.Arm` and `loomcli.ArmAt` share, reads the seed's recipe for the addressed run-id and picks the recipe's shed builder and routing from it for every verb that needs either: `BuildShed` for `step` and `run`, `Routing` for `goto`, `status` and `reject`, and the start and resume paths.
`loom` selects `loomrecipe.New` and `loomrecipe.Routing`, `darn` selects the darn builder and routing beside them.
So `lyx shed step` from the driver, `lyx loom resume` and `lyx loom goto Darn` over a darn seed all build the darn graph.
`reject` also takes its rework row from the recipe: `PR-Rework` in loom, `Darn` in darn.
It accepts a run awaiting or blocked at `PR-Gate`, or blocked at the rework row while a rejection record is pending, whose new rejection replaces the pending one; a darn run blocked at `Darn` with no rejection record is refused, since it has no pull request to reject.
Its refusal text names the recipe's two rows, never a row the run lacks.
The other `lyx loom` verbs (`approve`, `pause`, `commit-records`, `validate-description`) serve a darn run unchanged, since none builds a shed, reads routing or names a recipe row.

Rationale: the orch's run loop, PR-Gate procedure and recovery routes are written against `lyx batten` and `lyx loom` verbs; a second verb tree would duplicate `approve`, `reject`, `resume` and the driver launch for no new behavior.
The recipe is named darn everywhere a recipe is named: the seed, the board field, the config file, the stencil and the row.
The word loom in the verb names the module that drives task runs, not the recipe.

### D2. Batten seeds a darn child through the same bootstrap

Decision: batten's child seeding admits `loom` and `darn`, and the `Run-Shed` spawn stays `lyx loom start --no-attach` for both, since both recipes share that bootstrap verb.
The `parent` seed param is written for both recipes.
Any other name keeps today's unsupported-recipe refusal, whose text names the two admitted recipes.

Rationale: the refusal exists because the spawn is one bootstrap verb; admitting the second recipe that verb serves keeps the two consistent by construction.

### D3. The darn recipe is Preflight, Darn, Publish, PR-Gate, Finalize, Friction-Reflect

Decision: entry `Preflight`, terminal `Friction-Reflect`, rows in this order:

| Row | Engine | Segment | on_done | on_stuck |
|---|---|---|---|---|
| `Preflight` | `Preflight` | | `Darn` | |
| `Darn` | `DarnWrite` | `PR-Review` | `Publish` | |
| `Publish` | `Publish` | | `PR-Gate` | |
| `PR-Gate` | `PRGate` | `PR-Review` | `Finalize` | `Darn` |
| `Finalize` | `Finalize` | | `Friction-Reflect` | |
| `Friction-Reflect` | `FrictionReflect` | | `""` | |

`PR-Gate` keeps `max_bounces: 5`, the recipe literal loom uses.
`Darn` carries `segment: PR-Review` so the engine's same-segment rule admits the `PR-Gate` bounce.
No `Loom-Preflight` row: its batcher check is loom's, and the seed coherence it checks is loom's status seeder's own contract.
No `Describe` row: see D5.
`Darn` carries no `on_stuck`, so an exhausted gate halts the run blocked, the driver hands back and the parent is notified through the existing batten notice and driver report.
The spent-budget Stuck reason keeps the gate's findings and log path and names its way forward: run `lyx loom resume` in the task worktree, which re-spawns `Darn` with a fresh budget and the prior-work token (D4).
Routing edge and its bound: the `PR-Gate` to `Darn` edge re-spawns the writer with the rejection's findings.
`PR-Gate` returns Stuck only for an operator's matching `lyx loom reject`, so each traversal of the edge is one operator rejection, and that is the edge's whole bound.
`max_bounces: 5` does not cap it: a `Darn` Done ends `PR-Gate`'s bounce episode, since both rows share `PR-Review`, so the count never passes one, as with loom's `PR-Rework`.
Each traversal costs one `Darn` spawn with its own verify budget (D4).

Rationale: the operator decision names one producer, verify retries to a budget, PR-Gate as usual and escalation past the budget; the halt-for-a-human shape of a gated writer row with no `on_stuck` is already the recipe's convention.
The rejection edge needs no mechanical cap because a human starts every traversal.

### D4. `Darn` is a `SingleLLM` producer with a per-call spec, gated by `verify` then `description`

Decision: a `DarnWrite` registry entry in `internal/shedrecipe`, with its producer constructor beside `DiscussionWrite`'s in `internal/loomshed`, wraps `shedadapters.NewSingleLLMProducer` over a spec source evaluated at each call.
The stencil is `contracts/stencils/darn/darn-template-write.md`, registered in the stencil registry and seeded through `stencilstore`.
Its tokens are the slug, the description path, the parent and edit directives, the implementer pattern directive and the implementer friction directive, the rejection findings: the content of the rejection record when one exists for the run, else the literal `(none)`;
and the prior work: when the run's history holds an earlier `Darn` outcome or a Publish failure record is present, a Go-rendered note that the change is already under way on the task branch, naming the latest `Darn` outcome's reason and the Publish failure record's path, else the literal `(none)`.
The row's gates are `verify` then `description`.
The verify gate's attempts come from `darn.yaml`'s `verify_attempts` and are stamped onto the row by the darn recipe builder at every build (D1), the way the loom builder stamps the review budget from `loom.yaml`; the recipe row names the gate and the builder sets the budget.
The builder refuses a `verify_attempts` below 1 as its first act, as `loomrecipe.New` refuses a review budget below 1, so a value lowered after the start halts the next `step`, `run`, start or resume, the verbs that build the shed (D1), instead of reaching a gate with no attempts (D7).
`goto` builds no shed and darn's routing reads no `verify_attempts`, so a `goto` is not refused; the step after it is, and no gate runs outside a built shed.
The budget is per spawn: the gate counts failures per shuttle run, so a spawn after a rejection and a spawn after `lyx loom resume` of a halted `Darn` each start with the full `verify_attempts`.
Each spawn is started by an operator act, a rejection or a resume, which bounds the verifies a run makes.
The `description` gate keeps the recipe literal `attempts: 3`.
The writer's model and timeout come from `darn.yaml` through `shedrecipe.Env`.
The launch spec names the skills `scribe:prose` and `scribe:code-quality` and the color segment `darn`, added to `segmentcolor`'s set.
The agent name's role is `darn`.
The row's output file is the change description, and a commit decorator commits the landing directory after the gate loop settles, on Done and on a gate-failed Stuck alike, as `DiscussionWrite`'s decorator does.
The `Darn` row consumes the rejection record: on Done of a spawn that rendered one, it removes the record after the landing commit has landed, the order `PR-Rework` keeps, and a removal failure is a returned error.
A gate-failed Stuck keeps the record, so the spawn after a resume renders the same findings.
The row's interrupt policy is `reinvoke`.

The stencil tells the agent to: read the board entry with `lyx board get`; read `PATTERN.md` and the background files its change touches, through the rendered pattern directive; edit the repository; run package-scoped tests through `lyx gate test <pkg>` while working; commit on the task branch and never push; update the docs the repository's `CLAUDE.md` names for its change in the same commits; write the change description at the told path in the format `contracts/specs/final-summary-spec.md` defines, pointing at that spec and never restating it; run `lyx loom validate-description` before handing off; when the rejection token is not `(none)`, treat the findings as the work of this spawn;
and, when the prior-work token is not `(none)`, continue from the commits already on the task branch and fix what the token names, never restarting the change.
It states that the handoff runs the full verify and re-prompts with the findings, that a re-prompt is fixed in the same session with new commits, and that the way forward for a hub-wide refusal is the parent, never the operator.

Rationale: one session doing the whole change is the operator's definition of the recipe; the gate loop's in-session re-prompt is the mechanism that already sends a verify failure back to the same producer with the findings, so the retry budget is the gate's attempts.
A rejection becomes a fresh spawn because the previous session has ended at Publish; its findings travel as a token, the shape `Bouncer` uses for optional content.

### D5. The darn writer writes the change description; darn has no `Describe` row

Decision: the `Darn` session writes `_lyx/landing/`'s description file itself, gated by the existing `description` gate, and Publish and Finalize read it as they do today.

Rationale: the Describe session exists to read a long multi-generation run from the outside; a small change has one author who knows it, and the description gate pins the format either way.
One fewer LLM session per darn run.

### D6. The verify command is told; loom tells the plan's, darn tells `darn.yaml`'s

Decision: the landing producers already take `VerifyCommand` as a told closure; the verify gate constructor takes one too, and loom's wiring passes the plan reader while darn's passes a reader of `darn.yaml`'s `verify`.
Darn's reader loads `darn.yaml` at each call and returns an error, never `""`, when the file is missing or `verify` is empty, so the darn gate, Publish and Finalize each stop at call time instead of skipping their verify with a warning; a later `lyx config darn --set verify=` therefore halts the darn runs in flight rather than unguarding them.
The error is a loom refusal reachable through step, resume and goto, so its message names its way forward: run `lyx config darn --set verify=<command>` from the prime, then `lyx loom resume`.
The landing producers' wrap of a command-read error keeps the reader's message, so the way forward reaches the halt reason at all three sites.
The darn gate runs the comment lint from the task branch's merge base with the parent branch to HEAD, then the full verify command, with no impacted-set narrowing; a pass records the tree so Publish's own verify skips on the same tree and command.
While a Publish failure record is present the darn gate behaves as the loom round gate does: it also runs the failing tests the record names and, for a `publish_verify` failure, the `tmux` tier.
Publish's failed-verify Stuck reason takes its whole way-forward clause from the recipe through a told `Deps` field: loom's names the `Webster-Burler` goto and the Webster-Review round, as today; darn's is run `lyx loom goto --to Darn`, then `lyx loom resume`, in the task worktree, where the `Darn` spawn renders the failure record through its prior-work token (D4) and the darn gate runs the failing tests.
`PATTERN-verified-tree`'s site list gains the darn gate.

Rationale: a darn run has no plan, and the gate is the budgeted retry the operator asked for, so it must run the same verify Publish runs or the budget protects nothing.
The lint base is the merge base because the darn's commits are the whole diff of the run.

### D7. `darn.yaml` is a hub-wide config module

Decision: a `darn` module in `configreg`, registered like `landing` (hub-wide, read at `<BoardDir>/_lyx/config/darn.yaml`), with these keys and template defaults:

| Key | Default | Meaning |
|---|---|---|
| `writer` | `opus[medium]` | model-spec of the `Darn` session |
| `writer_timeout_min` | `180` | minutes the session may run |
| `verify` | `""` | the full verify shell command, run in the task worktree by the darn gate, Publish and Finalize |
| `verify_attempts` | `3` | re-prompts the verify gate gives the session before the run halts |

`lyx loom start` on a darn seed loads the file strictly, reads `verify` through D6's reader and builds the darn shed, so it refuses when `verify` is missing or empty or `verify_attempts` is below 1.
These are two refusals (D13):
the empty `verify` is refused by D6's reader at start and at each call of the darn gate, Publish and Finalize, with the way forward `lyx config darn --set verify=<command>` from the prime, then `lyx loom resume` for a run past its start;
a `verify_attempts` below 1 is refused by the darn builder at every shed build (D4), with the way forward `lyx config darn --set verify_attempts=<n>` from the prime, then `lyx loom resume`.
The start refusal is the early check; the reader and the builder are the guards that hold for every later step and resume.
The `verify` key is the same trust class as landing's `publish_verify`: it runs any shell command in the task worktree, which is why it is hub-wide and never read from a pair's own `_lyx/config/`.

Rationale: the verify command is a repository fact, and a task must not be able to weaken its own gate through its tree, the rule landing's `publish_verify` already states; the writer model and the budget are how this hub runs darns, the same class as landing's `describe` and `conflict` models.
An empty verify would make the gate pass on a warning, which is the loom behavior for a plan without `## verify:` and wrong for a recipe whose only safety net is that gate.

### D8. A verify log per run, named by sequence, with a bounded history

Decision: `verifytree.Verify` writes each run's output to `verify-<n>.log` in the verify directory, `n` one above the highest existing number, and reports the path on its result; the fixed `verify.log` name is retired.
The Publish failure record names the run's own log instead of copying it.
After writing, Verify removes logs beyond the newest twenty unless a Publish failure record names them.
The marker and the verified-tree record are unchanged.

Rationale: #502's loss was a later verify overwriting the evidence of an earlier failure; one file per run removes the overwrite, and the retention bound keeps a long-lived prime's directory from growing without end.
The evidence that must outlive pruning is the failing Publish verify's log, and the failure record pins it; twenty keeps the recent gate attempts around it for a reader, and does not claim to hold every verify of a run.
The record is one file per verify directory naming one log, so it pins at most one log beyond the twenty.

### D9. Finalize commits the parent weft's run records before its parent-side merge

Decision: Finalize, after its catch-up merge-in and before opening the parent pair for the parent-side merge, commits the parent weft's own `_lyx/shed/` subtree through a told landing seam over `fabricengine.CommitAnchoredPaths` on the parent location, with a commit message naming the landing slug; a clean subtree is a no-op.
A commit failure is Stuck naming the parent pair and the way forward `lyx fabric commit` run in the parent pair, then `lyx loom resume`.
Every text `landingshed` produces for this step, Stuck reason, log line and commit message, names the parent pair and its run records, never a side, per `PATTERN-fabric-vocabulary`; the side lives only in the told seam's `fabricengine` call.
The merge guards are not weakened: the clean-pair check keeps both sides.

Rationale: the parent-side merge's pre-merge sync fast-forwards both checkouts and refuses a dirty one; the dirt is batten's own durable status records of sibling runs, written on every transition and committed only at the next one, and skipped altogether while the parent pair is mid-merge.
Committing them is what batten does next anyway, so landing them early loses nothing and lets the guard stay strict.
Batten's no-op-transition marker still skips a pair the seam already committed, and its next commit over a clean path is a no-op.

### D10. Pair teardown settles the run's board claim

Decision: `internal/pairteardown` settles the board entry's run-form status after a successful removal, judging the run as well as the board.
A done entry and an absent entry are left alone, an entry whose status lacks the run form is left alone, and a board or status-read failure only warns.
Otherwise the outcome follows the run:

- Landed: the run is landed when the teardown request carries a landed flag, which batten's `Worktree-Teardown` row sets since that row runs only after a done child, or when the child run's own status, read from the task worktree before removal, holds a `Finalize` Done in its history.
  A landed teardown marks the entry done, never clears it, so a Finalize whose own done mark failed still ends with a done entry on every route, including a hand removal after the run halted at `Friction-Reflect`.
- Not landed, the `lyx fabric remove` route: the composite reads the prime's batten run for the slug.
  When that run is `running`, the entry is left alone with a warning naming `lyx batten status <slug>`, since the batten run still holds it.
  Batten stays `running` while its child halts, since `Run-Shed` waits on a halted child without ending, so a darn halted on a spent verify budget keeps its claim here; the warning names `lyx batten pause` as the step that turns it into an abandon.
  When that run is `done`, the entry is marked done.
  When the slug has no batten seed, or batten itself is halted (`paused`, `blocked`, `failed`, `awaiting`), the status is cleared to empty: removing the pair by hand is the operator's decision to abandon that run, the condition `RunLockedError` already names for clearing.

What the clearing lets through: scope edits on an entry whose batten run is halted or that was never seeded through batten, and one landed case, a run whose Finalize merged and failed its done mark but returned Stuck on its push, so its history holds no `Finalize` Done.
What bounds it: it never fires while a batten run for the slug is `running`, never on a run whose history holds a `Finalize` Done, and only after the composite has ended the pair's session and removed both worktrees, so no step of the run can still write.
The one landed case needs two failures in one Finalize, and its change is on the parent branch, where the operator re-marks the entry done with `lyx board set-status`.
The composite performs this itself with the hub path it already holds, so every teardown route releases the claim by construction.
`PATTERN-pair-teardown` gains the clause.

Rationale: `board-lock-running` refuses scope changes while the status has the run form, and nothing clears the form when a run ends without Finalize marking the entry done; the chokepoint every teardown passes is the one place to do it.
Clearing to empty returns the entry to the state it had before the run claimed it, and batten's notices already tell the orch why the run ended.
The batten-state test is the one `RunLockedError` tells a human to run before clearing by hand, so the composite applies the same rule.

### D11. The halt note carries the parent pair's fabric status when a halt names a dirty worktree

Decision: when a loom halt's reason contains fabric's dirty-worktree reason, the Go-authored `loom-halt` note gets a `parent fabric status` section listing the parent pair's dirty tracked paths, read in-process through `fabricengine`'s status over the parent location; a read failure writes the error in the section.
Each path is written as an absolute path under its worktree, with no side label, so the section names the tree without `internal/loomcli` spelling a side, per `PATTERN-fabric-vocabulary`.
The reflection stencil gains one sentence: a halt note with that section attributes a dirty-tree halt to the tree the section names.
No `lyx fabric status` runs in a stencil.

Rationale: `PATTERN-fabric-git` keeps git in fabric, and principle 7 keeps a deterministic read in Go; the agent only needs to read what Go already gathered.

### D12. The orch stencil, board skill and board help name the recipe

Decision: the orch role stencil's run loop says the batten run drives the task's run, loom or darn, and that a darn run has no parent review and no circling, PR-Gate being its one review point, where a reject re-spawns the darn writer with the findings; the `ly:board` skill and the board CLI's `recipe` help name `loom` and `darn` as the values;
`lyx loom reject`'s help names both recipes' rework rows: in loom the next start sends the findings to `PR-Rework`, which archives the plan generation, and in darn to a fresh `Darn` spawn.

Rationale: `PATTERN-no-denied-recovery` and the orch's recovery routes are the same for both recipes, so the stencil needs only the one difference.

### D13. Loom's refusal-spec rows this change adds or edits

Decision: the loom section of `contracts/specs/refusal-spec.md` changes in these rows, each message naming the way forward its row records and each reached by a test:

| Row | Change | Way forward |
|---|---|---|
| `darn.yaml`: `verify` missing or empty | new | `lyx config darn --set verify=<command>` from the prime, then `lyx loom resume` for a run past its start |
| `darn.yaml`: `verify_attempts` below 1 | new | `lyx config darn --set verify_attempts=<n>` from the prime, then `lyx loom resume` |
| `Darn` verify gate exhausted | new | `lyx loom resume` in the task worktree, which re-spawns `Darn` with a fresh budget (D3) |
| Finalize: parent run records commit failed | new | `lyx fabric commit` in the parent pair, then `lyx loom resume` (D9) |
| Publish: verify failed | edited | the Way forward cell gains darn's clause beside loom's (D6) |
| gate slot: gate.yaml absent or invalid | edited | the trigger names the darn gate among the sites; the way forward is unchanged |

Rationale: `PATTERN-refusal-way-forward` binds every refusal a loom verb reaches, not only the new config keys, and the spec's rule keeps each Way forward cell equal to its message.

## Constraints

- `PATTERN-shed-recipe-registry`: `DarnWrite` is one more constructor in `internal/shedrecipe`'s map, with no `init()` and no `lyxcwd`.
- `PATTERN-shed-verb-set`: `darn` is one more name in `internal/shedcli`'s single map literal, armed by `loomcli.ArmAt`; no verb body is reimplemented.
- `PATTERN-shed-run-directory`: `darn` is added to `shedrun`'s closed recipe vocabulary, the sole declarer.
- `PATTERN-driver-choice-single-site`: untouched; loom's arm reads the seed's recipe field (D1), and the driver field gains no reader.
- `PATTERN-told-geometry`: the darn entry and the verify gate take told roots and closures; `darn.yaml` is read in `loomcli`'s wiring and handed down.
- `PATTERN-hub-wide-config`: `darn` is marked `HubWide` and read at the board dir only.
- `PATTERN-config-strictness`: `darn.yaml` loads through `configengine.Load`, strict.
- `PATTERN-stencil-ownership`, `PATTERN-role-skills-typed`, `PATTERN-parent-directive`, `PATTERN-edit-directive`, `PATTERN-spawn-color`: the darn stencil renders the parent and edit directives, the launch spec names the skills and the color segment, and the stencil asks for neither.
- `PATTERN-producer-pointer-rule`: the stencil points at `final-summary-spec.md` and `refusal-spec.md`.
- `PATTERN-no-denied-recovery`: no refusal, stencil or spec text added here names a denied form or a raw module-wide `go` run; tests run through `lyx gate test`.
- `PATTERN-refusal-way-forward`: every row D13 adds or edits lands with its message and a test that reaches it, the two `darn.yaml` refusals at start and after it.
- `PATTERN-verified-tree`: the darn gate joins the site list; the per-run log and the failure record's log reference replace the "copy of the log" clause; Publish still alone writes the failure record.
- `PATTERN-fabric-git`: D9's commit and D11's status read go through `fabricengine`.
- `PATTERN-pair-teardown`: the composite stays the sole route and gains the claim clause.
- `PATTERN-friction-capture`: the halt note stays Go-authored and the reflection still files only through `lyx selfreport create`.
- `PATTERN-review-round`, `PATTERN-batcher-registry`, `PATTERN-plan-generation`: not touched; darn has no review segment, no batch and no plan.
- `PATTERN-sandbox-coverage`: the `darn` config module is exercised by a sandbox scenario or excluded with a reason.
- `PATTERN-test-economy`, `PATTERN-test-speed`, `PATTERN-test-isolation`: new behavior is pinned at the public surface, offline with fakes where a fake can see it, and `cmd/lyx/testdata/test-budget.yaml` rows are raised in the same commit where a package's count needs it.
- `PATTERN-comment-line-breaks`, `PATTERN-markdown-link-integrity`: all prose follows them.
- `PATTERN-fabric-vocabulary`: `landingshed`, `battenshed`, `pairteardown` and `loomcli` keep naming "the parent pair" and "the task worktree", never a side, in code, messages and the halt note (D9, D11).
- CLAUDE.md: docs land in the same commit; no versioning; the recipe is never called loom.

## Auto-mode assumptions

- The CLI stays `lyx loom` for darn runs (D1); if the operator wants `lyx darn` aliases, that is a later board entry.
- The darn writer defaults to `opus[medium]` with a 180-minute timeout (D7).
- The description is written by the darn writer rather than a Describe row (D5).
- `darn.yaml` is hub-wide with no per-pair override (D7).
- Twenty is the verify-log retention count (D8).
- The board claim is cleared to empty rather than set to a word (D10).
- A hand removal of a pair whose batten run is halted abandons that run and clears its claim (D10).
- The darn gate runs the full verify command and does not narrow by impacted set (D6).
- The darn gate's `tmux`-tier confirmation while a Publish failure record is present follows the loom round gate's existing rule (D6).

## Open risks

- A hub whose `darn.yaml` does not yet exist refuses the first darn start until the operator sets `verify`; the refusal names the command.
- `lyx loom start` from a worktree with no seed still seeds `loom`, so a darn run can only start through batten; a hand-started darn is not supported.
- Clearing the claim at teardown also fires for a pair removed by hand while its batten run is halted; that is the intended release, but the entry loses the trace of where the run stood, which batten's own `_lyx/shed/<slug>/` records in the prime keep.
- A pair removed by hand while its batten run is `running` keeps the claim; the operator settles that run through `lyx batten status` and the existing routes.
- Committing the parent weft's records from Finalize attributes a sibling run's status commit to the landing run's message; the content is batten's and is unchanged.
- The `Darn` row sits in segment `PR-Review` from the first step, so a status reader that labels rows by segment shows a writer under a review segment name.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 3: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lyx/reviews/discussion/round-3-review.md; fixer report: _lyx/reviews/discussion/round-3-fixer-report.md.
  - d6-call-time-stop-no-way-forward-or-spec-row (class design, severity BLOCKING, open in rounds 2, 3)
  - darn-reentry-input-missing (class design, severity MEDIUM, open in rounds 3)
<!-- /lyx:carry-over Discussion-Review -->

<!-- lyx:carry-over Plan-Review -->
- Plan-Review, round 2: converged; each finding below was addressed by the round's fixer, and no fresh reviewer has seen the fix.
  Review: _lyx/reviews/plan/round-2-review.md; fixer report: _lyx/reviews/plan/round-2-fixer-report.md.
  - card3-gone-worktree-claim-unsettled (class design, severity MEDIUM, open in rounds 2)
  - card3-real-claim-substrates-untested (class scope, severity MEDIUM, open in rounds 2)
  - darn-seed-bound-card13-vs-card12 (class consistency, severity MEDIUM, open in rounds 2)
<!-- /lyx:carry-over Plan-Review -->

## Acceptance criteria

Recipe and selection:

- The embedded darn recipe parses and builds against the registry, its row names match pinned constants, every row has an interrupt policy, and `PR-Gate`'s `on_stuck` resolves to `Darn` within segment `PR-Review`.
- A board task with `recipe: darn` is seeded by batten with a `darn` child seed and the `parent` param, and `Run-Shed` starts it through the shared bootstrap; a task with an unknown recipe keeps the unsupported-recipe refusal naming `loom` and `darn`.
- `lyx loom start` over a `darn` seed builds the darn shed and keeps the seed; over no seed it seeds `loom`.
- `lyx shed step` and the four generic verbs arm a `darn` run through `loomcli.ArmAt`.
- Over a darn seed, `lyx shed step`, `lyx loom resume` and `lyx loom goto Darn` each build the darn shed and accept `Darn` as a row; over a loom seed they build loom's.

The `Darn` row:

- A gate-failed verify re-prompts the live session with the findings and the log path; the attempts equal `darn.yaml`'s `verify_attempts`, counted afresh for each spawn; a spent budget returns Stuck with the gate's reason and the way forward `lyx loom resume`, and the run halts blocked at `Darn`.
- The spec's rejection token is `(none)` with no rejection record and the record's findings with one.
- The spec's prior-work token is `(none)` on a first spawn, and names the latest `Darn` outcome after a resume of a spent budget and the Publish failure record after a goto from a Publish verify failure.
- `lyx loom reject` accepts a darn run at `PR-Gate` and a darn run blocked at `Darn` with a pending rejection record, replacing it, and refuses a darn run blocked at `Darn` with none, its message naming `PR-Gate` and `Darn`.
- A Done of a spawn that rendered a rejection removes the record after the landing commit; a gate-failed Stuck keeps it.
- The description gate's findings re-prompt the session, and a Done or gate-failed outcome commits the landing directory.
- The launch spec carries the two skills, the `darn` color segment and the `darn` role; the stencil renders the parent, edit, pattern and friction directives and names no denied form.

Config:

- `darn.yaml` reconciles from its template, is listed hub-wide, and an empty `verify` or a `verify_attempts` below 1 refuses `lyx loom start` with the recorded way forward; each refusal is reached by a test and has its spec row.
- With `verify` emptied after the start, the darn gate, Publish and Finalize each stop on the reader's error, whose halt reason names `lyx config darn --set verify=<command>` and `lyx loom resume`, and none skips its verify.
- With `verify_attempts` lowered below 1 after the start, the next `lyx shed step` refuses at the darn builder with the recorded way forward, and no gate runs.

Verify:

- The darn gate runs the comment lint from the merge base, then the told command in full, records the pass, and Publish then skips on the same tree and command.
- With a Publish failure record present, the darn gate runs the record's failing tests, and for a `publish_verify` failure also the `tmux` tier as the loom round gate does.
- Publish's failed-verify Stuck reason names the `Darn` goto and the darn gate in a darn run, and the `Webster-Burler` goto and the Webster-Review round in a loom run.
- Two consecutive verifies in one worktree leave two logs, each result naming its own; the failure record names the failing run's log; the twenty-first log prunes the oldest unless a failure record names it.

Finalize and teardown:

- A dirty `_lyx/shed/` subtree on the parent weft is committed before the parent-side merge and the landing proceeds; a dirty path elsewhere on the parent still refuses as today; a commit failure is Stuck with the recorded way forward.
- Batten's teardown row over a run-form entry marks it done; a done entry and an absent entry are unchanged.
- `lyx fabric remove` over a run-form entry clears the status when the slug has no batten seed or its batten run is halted, marks it done when that run is `done`, and leaves it with a warning when that run is `running`.
- `lyx fabric remove` over a run-form entry whose child history holds a `Finalize` Done marks it done, with batten `paused`, `failed` or absent.
- A board or status-read failure does not change the teardown's verdict.

Reflection:

- A halt whose reason names a dirty worktree writes a halt note with the parent pair's dirty paths; a halt with another reason writes none; a status read failure writes the error in the section.

Docs and guards:

- `PATTERN-verified-tree` and `PATTERN-pair-teardown` carry their new clauses; `docs/overview.md`'s loom bullet and execution stack name the darn recipe; the loom section of `refusal-spec.md` holds D13's four new rows and its two edited ones; `shed-recipe-spec.md`'s gate vocabulary lists `verify`.
- The orch role stencil names the loom and darn recipes and darn's single PR-Gate review point, the `ly:board` skill and the board CLI's `recipe` help list `loom` and `darn`, and `lyx loom reject`'s help names both recipes' rework rows.
- The sandbox coverage guard passes with `darn` covered or excluded with a reason, and every test-budget row holds.

## Notes for the plan writer

- Precedents to mirror: `internal/shedrecipe/entries_discussionwrite.go` with `internal/loomshed/discussionwrite.go` for the gated, committing single-LLM row; `internal/loomrecipe`'s budget stamping for the gate attempts; `internal/landingshed/config.go` and its `configreg` registration for the hub-wide module; `internal/loomcli/landingdeps.go`'s `VerifyCommand` closure for the told command.
- The shed CLI recipe table is in `internal/shedcli/table.go`; the seed vocabulary in `internal/shedrun/seed.go`; batten's admit list in `internal/battencli/wire.go`'s `WriteSeed` and `childSeedParams`.
- The verify gate today is `loomshed.NewVerifyGate`, plan-bound; the darn gate is either a told-command variant of it or a sibling sharing its findings renderers.
- Publish's goto text is in `internal/landingshed/verifygate.go`; the row name becomes a `Deps` field.
- `pairteardown.Teardown` already holds the prime location, which gives the hub path for the board.
- The halt note writer is `internal/loomcli/halt.go`; fabric's dirty reason constant is in `internal/fabricengine/mergeerrors.go`.
- The `tmux` tier is run by `publish_verify` at Publish for darn exactly as for loom; nothing changes there.
