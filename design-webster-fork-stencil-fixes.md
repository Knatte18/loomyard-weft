# Webster fork and pattern-directive stencil fixes from the stencil intent audit

`webster-fork-stencil-fixes` · Task · bug, stencil, webster, burler, loom · running · Discussion-Write

Bundle of the Webster fork and pattern-directive findings of the stencil intent audit of 2026-10-15, all in `contracts/stencils/webster/`, `contracts/stencils/pattern/`, `contracts/stencils/edit/`, the Webster-Review rubric and the plan spec.
Line numbers as of `main` on 2026-10-16.

## A `Rename` uses a structure-aware tool (F7, was `webster-rename-script`)

### Seen

- `contracts/stencils/edit/edit-directive.md:6` bans every script that rewrites files, `python` and `node` heredocs included.
- `contracts/specs/loom-plan-spec.md:126` and `:335` make a `Rename` of a symbol an AST-aware script plus a grep, never a text or regex replace, and `:335` names a library per language.
- `contracts/stencils/loom/loom-rubric-webster-review.md:79` asks the reviewer to confirm that check ran and passed, which a diff cannot show.
- The webster fork prompt (`webster-body-implementer.md`) never tells a fork to run such a tool, so a fork obeys the edit directive and renames by hand.
- The plan's canonical `## Rename mechanic` section (`contracts/specs/loom-plan-spec.md:202-215`, reproduced by `loom/loom-template-plan.md:209-220`), which the master reads and the forks inherit, prescribes `git mv` plus surgical edits: a file move, while a `Rename` card's pairs are symbol renames.

### Decision

Operator decision (2026-10-16): the AST-aware rename tool is required for a `Rename` in Webster and stays.
It is the one exception to the edit directive, and only for this part of Webster.

Operator decision (2026-10-16), replacing the earlier Python-fallback wording:

- The stencil names only the rule: a symbol-level rename uses a structure-aware tool, never text search-and-replace.
- The concrete tool comes from the hub's language setup; no Go or Python tool is written into the stencil.
- Python is allowed where the hub has turned off shuttle's `deny_python` switch (`internal/shellguard/rows.go:20`), such as a Python repository; lyx serves hubs on other machines and repositories in other languages, so a hub may need it.

### Idea

Agent's suggestions:

- The edit directive names the exception once: a fork working a `Rename` card may run the hub's structure-aware rename tool, and nothing else may.
- The fork prompt tells a fork on a `Rename` card to use the tool the hub's language setup names, then the grep pass.
- Webster-Review checks the rename's outcome, which a diff shows: no reference to the old name left and the symbol's doc comment updated, instead of "the tool ran".
- The spec's Rename row, its per-language list at `:335` and the `## Rename mechanic` text are updated to match, and the mechanic section separates a file move (`git mv` plus surgical edits) from a symbol rename (the structure-aware tool).

## A fork's departure reaches the Master through the commit (F12 and F13, was `fork-deviation-in-commit`)

### Seen

A webster fork has no channel for two things:

- `webster-body-implementer.md:98` has it name a failure its card did not cause "in your final reply to Master", but Master never reads raw fork output (`webster-template-master.md:103`, `:246-247`) and the batch report holds only `status` and `head_sha` (`webster-body-implementer.md:104`, which also says Master reads that report, when Go reads it through `record-batch`).
- `contracts/stencils/pattern/pattern-directive-implementer.md:19` has an implementer that meets a PATTERN conflict follow PATTERN and "say so in your report", which the same two-field report cannot hold.

An in-session fork never receives the implementer directive: `RenderForkPrompt` renders no pattern directive, and the fork inherits the master's orchestrator directive (`internal/websterengine/render.go:626`), whose `pattern/pattern-directive-orchestrator.md:19` holds the fork's only PATTERN rule.
The implementer directive reaches the recovery strand, the plan writer, the PR-Rework writer and the darn writer.

A fork that follows PATTERN over its card leaves a diff that departs from the plan with no stated reason, so Webster-Review, which measures the diff against the plan, flags the departure and its fixer reverts it into the PATTERN violation.

The verify-fix fork is the one fork whose reply the master is told to read: `webster-template-master.md:171` has the summary name "what the fixer's reply says it changed", against `:103` and `:246-247`.

`webster-body-implementer.md:98` says a failure the card did not cause is "never a `FAILED` on its own", while `:111-113` make a failed batch gate `FAILED`, so a fork whose batch gate fails for a reason its card did not cause has no consistent status.

### Decision

Operator decision (2026-10-16), on the agent's recommendation:

- A fork that departs from its card because of a PATTERN entry states the departure and the entry in that card's commit message body.
- Webster-Review's rubric reads the commit messages of its range as well as the diff, and takes a departure stated there as the reason, not as an unexplained mismatch.
- The "name it in your final reply to Master" sentence goes: a failure the card did not cause is left to the plan-level verify gate, which runs the whole suite and spawns a fixer.
- No new report field.

Operator decision (2026-10-16): a reported departure is meant to reach the Master, so that every later fork gets it through the context it inherits.

- `record-batch` extracts the departures stated in the batch's commit bodies into the batch digest the Master receives, so the Master holds them and later forks inherit them.
- The Master still never reads raw fork output; the departure reaches it through Go.

### Idea

Agent's suggestions:

- Put the departure rule where the in-session fork reads it: `webster-body-implementer.md`, which every webster fork and the recovery strand get, and `pattern-directive-orchestrator.md:19`.
- A change to `pattern-directive-implementer.md:19` covers all its readers: the recovery strand at the commit message, and the plan, PR-Rework and darn writers at their own outputs (the plan's `## Shared Decisions` for the plan writer).
- Correct `webster-body-implementer.md:104` to say `record-batch` reads the two fields.
- The master's summary reads the verify-fix fork's `fix:` commits instead of its reply, so `:171` agrees with `:103` and `:246-247`.
- State the status a fork gives when its batch gate fails for a reason its card did not cause, so `:98` and `:111-113` agree.

## Fork spawn and completion wording (F5, was `webster-fork-spawn-wording`)

### Seen

- `contracts/stencils/webster/webster-prefix-fork.md:24` tells the fork that "Master's own `await-batch` sees it and takes over" once the report is written.
  Master no longer runs `await-batch`: it ends its turn after the spawn and is woken by the fork's completion notification (`internal/websterengine/doc.go`).
  The banner at `:3` likewise names an Agent-tool call, "Read this file and follow it exactly: <path>", that Master never sends.
- `webster-template-master.md:12` tells a fork to recognise itself by that same instruction, but the fork prompt Master sends (`:73`, and the verify-fix fork at `:168`) reads "Read this file and do exactly and only what it says".
- `webster-template-master.md:188-189` says a stuck run resumes "once an operator has looked at the plan", with the operator editing the plan and running `rebaseline`; under loom the run is routed to plan review, and the orch never edits `_lyx/plan` (`orch-template-role.md:99`).

### Decision

Operator decision (2026-10-16): the stencils' account of how a fork is spawned and how Master learns it finished is wrong and is to be fixed; the operator leaves the shape of the fix to the agent.

Operator view, not a decision: Master's prompt cache is believed to stay warm while a fork runs.

### Idea

Agent's suggestions:

- In the fork prefix, say the fork's turn ends after the report and Master is woken by its completion notification, and drop the `await-batch` wording except the ban on running it.
- Make the master's self-identification rule at `:12` quote the instruction Master sends, or key it on "You are an implementer fork", which both spawn prompts open with.
- Reword `:188-189` to name who looks at the plan under loom (plan review) instead of "an operator".
- The cache remark needs no change here: the fix touches only wording.

## Drop the inline kick-start pack layout (F3, was `webster-drop-inline-pack`)

### Seen

Webster's kick-start pack has three layouts: per card (`kickstart_pack_per_card`, the default in a hub run), a pack file (`kickstart_pack_file`, the standalone fallback) and inline in the fork prompt (both keys false).
The inline layout's fragments are frozen as a measurement baseline (`contracts/stencils/webster/webster-fragment-fresh-read-pack-inline.md:4`, `webster-fragment-locate-pack-inline.md:4`, `webster-fragment-pack-section-inline.md:4`), so `webster-fragment-fresh-read-pack-inline.md:8` still says "one Master read before forking you", which `webster-master-no-code` made untrue for the other layouts.
`tools/forkaudit` has an `inline` arm for that layout.

### Decision

Operator decision (2026-10-16): code text is never pasted into a prompt without a good reason, and the reason is written down.
Inlined text goes stale as the batch's cards change the tree, and it gives the agent no sign of what it has already read from the source.

### Idea

Agent's suggestions:

- Remove the inline layout, its three fragments and the measurement-baseline freeze, so `kickstart_pack_file: false` with `kickstart_pack_per_card: false` no longer selects it; the per-card layout and the pack-file fallback stay.
- The pack-file layout stays because it holds pointers, not code: the fork reads the files itself and sees what it has read.
- `tools/forkaudit`'s `inline` arm is kept for reading old sessions.
- `review-inline-code` proposes pasting cited code into review rounds and must state its good reason against this decision.

## A task that changes PATTERN declares it (F4, was `pattern-change-declared`)

### Seen

Every role that gets the pattern directive (plan writer, Plan-Review, the Webster master and so its forks, the burler reviewer, the Bouncer judge, the darn writer, the PR-Rework writer and the Discussion designer) gets `PATTERN.md` as it stands at the worktree root when its prompt is rendered.
The reviewer and judge directives make every cited violation BLOCKING and never downgradable (`contracts/stencils/pattern/pattern-directive-review-fix.md:18`, `pattern-directive-judge.md:17`).
This repository's `CLAUDE.md` puts a new invariant into `PATTERN` in the same commit as its code, so a task that changes a PATTERN entry is judged against the old text: a reviewer flags the very change the task is meant to make as a violation.
The webster master reads `PATTERN.md` and the `pattern/` files once and its forks inherit them; `webster-template-master.md:31` claims those reads "stay current for the whole run because the run refuses a drifted plan", which covers the plan but not a card that edits `PATTERN.md`.
The fresh-read fragments already tell forks to read a targeted rules file fresh (`webster-fragment-fresh-read-plain.md:6` and the pack variants), so the staleness half is covered for forks, leaving the master's claim at `:31`.
The Discussion already names each entry a design changes (`pattern-directive-designer.md:18`), but nothing carries that to the later roles as a sanctioned change.

### Decision

Operator decision (2026-10-16): a task that changes PATTERN says so explicitly in its decision record, as a clear "this changes PATTERN" statement naming each entry and its new rule, so every reviewer knows.
The edit itself still happens in Webster, not before.
The plan carries the same statement, so its reviewers raise no conflict either.

Operator view, not a decision: Plan-Review reads the decision record anyway.
(Agent's check: it does, as its answer key; Webster-Review measures against the plan only, so the plan's copy is what reaches it.)

Open question: whether the same treatment applies to `CLAUDE.md`, which cards also edit; the operator has not decided that.

### Idea

Agent's suggestions:

- A fixed section in the decision record, defined in `loom-discussion-spec.md` and checked by `validate-discussion`, listing each PATTERN entry the task adds, changes or removes with its new text.
- A matching plan-level section in `00-overview.md`, required by the plan gate whenever the record's section is non-empty.
- The reviewer, judge, implementer and orchestrator pattern directives say that an entry the task lists as changed is judged against the listed new text, never the old.
- The webster master's claim at `:31` is narrowed to what the drift check covers.
- This section and the departure rule above both edit `pattern-directive-implementer.md:19` and must agree on which PATTERN text a fork follows when the task declares a change to that entry.
