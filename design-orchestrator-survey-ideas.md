# Remaining leads from the survey of comparable agent orchestrators (Oct 2026)

`orchestrator-survey-ideas` · Note · feature, burler, shed, orch, quarry, report

Leads borrowed from comparable open-source agent orchestrators, surveyed in October 2026 (first imported from issue #671).
Nothing here is a commitment; these are leads to triage.
The orch checked each against the code on 2026-11-17: ideas lym has built or that proved moot are dropped,
partly built ones name only what is still open, and the eight most worth doing are notes of their own in this project.
Each bullet names its source project and the lym module it would touch.

## Plan validation and concurrency (quarry, Plan-Write, board)

- **Typed scope operations on cards** (foremerge): each scope carries an additive or breaking operation,
  and deterministic rules flag colliding pairs even when files do not overlap.
  Worth comparing against board labels and the alignment gate.
- **Symbol leases between concurrent tasks** (Fusion, ORCH): expiring leases on symbol keys serialize overlapping tasks,
  with a deterministic tie-break; lym keeps overlapping tasks apart by board labels.
- **Commit-time reservation guard** (ntm): a pre-commit hook refuses a commit touching what another task holds; needs leases first.
- **Criterion amendment ledger** (oh-my-claudecode): a card's acceptance text changes only through a recorded amendment in weft.

## Review (burler, Bouncer)

- **Quorum of differently-angled judges** (ccswarm): lym fans out lenses but has no quorum vote for high-risk segments.
  Since 2026-11-19 a run's parent orch can be asked to see and veto each Code-Review round and each drift judge ruling
  (`code_review_ask_parent`, `judge_ask_parent`); that is one extra judge, not a quorum.
- **Risk tier by changed paths** (yylo): globs over sensitive paths choose the review depth and the gate scope.

## Engine, recipes and governance (shed, tender, orch, board)

- **Cross-run quarantine** (Bernstein): a task that fails three times across runs is held, unless the failures were the host's.
- **Fork a run from a step with a tree snapshot** (Bernstein): `shed goto` moves the cursor but does not restore the warp tree.
- **Warm retry guarded by a workspace hash** (Bernstein): resume the native session only when the tree is unchanged.
- **Evidence reuse by input identity** (yylo): verify is skipped on the same tree; a changed review prompt could rerun only the review.
- **Guidance at the next step boundary** (Conductor): a queue of operator guidance for the next producer's prompt.
  Partly met since 2026-11-19: `notify_parent_on` in `loom.yaml` mails the run's parent when a named row is reached,
  so the parent can brief that row's agent as it starts; there is still no queue of guidance lym itself folds into the prompt.
- **Batched operator questions with recorded assumptions** (Spec Kit `/clarify`, Conductor): Discussion-Write asks at most a few ranked questions,
  each with a recommended answer, and records unanswered ones as assumptions.
  Since 2026-11-19 a run's autonomous agents put their questions to the run's parent orch, not to the operator in their pane.
  Discussion asks in its pane only when `discussion_interactive` is `true`, which is off by default and forced off by autonomy mode (`30ed1ee58`),
  so this lead concerns an interactive Discussion only.
- **Rate-limit wait**: the orch watcher wakes the orch after a usage reset; tender itself schedules no resume.
- **Standing goals and scheduled routines** (ORCH, CAO, Fusion): periodic triage, dependency bumps or `selfreport` sweeps.

## Agent sessions (reed, shuttle)

- **Waiting-for-input signal** (agent-of-empires): shuttle has an awaiting-input state; screen heuristics still decide readiness and startup.
- **Reference status patterns and a paste pitfall** (CAO): compare its regexes and its tmux bracketed-paste note with shuttle's readiness and creel inject.
- **Context forecasting** (ntm): the orch cycles its context; Webster's Master and the loom driver do not, and nothing forecasts.
- **Staggered spawns** (ntm): space Webster forks and batten fan-out by learned rate limits.

## Observability (report, weft)

- **Hash-chained run journal** (Bernstein): a hash chain over producer outputs makes an archived run self-verifying.
- **Unverified-completion rate** (Bernstein): a `lym report` view of completions with no test or gate behind them.
- **Commit-to-record links** (AgentNote): weft links to warp by a `Warp-SHA:` trailer; a note or trailer on each warp commit could link back.
- **Run diff** (ccswarm): `lym report` comparing two archived runs' timelines and token use.
- **OpenTelemetry spans per producer** (Conductor).

## Operator attention (orch, dobby, reed)

- **Attention feed with acknowledgement states** (ntm): dobby's wait reminders have no seen, snoozed or acknowledged states.

## Discussion review (Discussion-Write, stencils, pattern)

- **Requirements-quality checklist** (Spec Kit `/checklist`): items with IDs the Discussion-review judge answers one by one.
- **Fixed analysis categories** (Kiro): inconsistency, ambiguity, conflicts, unstated assumptions and missing edge cases as named lenses.
- **PATTERN rules handed to reviewers** (Spec Kit constitution): its rules as numbered MUSTs in plan and code review prompts.
- **Glob-scoped steering** (Kiro): stencils pull per-area guidance chosen by the paths of a plan's glyphs.
