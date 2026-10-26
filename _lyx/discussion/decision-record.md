# Decision record — webster-tree-moved-fixer

## Goal

A `begin-batch` refusal with `plan_conflict` whose findings are mechanical, such as a card naming a symbol or path an earlier batch renamed, is fixed inside the Webster run by a small cold plan-fixer strand that edits the named cards and rebaselines them, and Webster goes on.
Only a conflict whose fix changes what a card does still ends Webster stuck, and under loom still reroutes to Plan-Burler-Review.

## Scope

- A new role-audience verb `lyx webster fix-plan`, modelled on `lyx webster fix-verify`: it spawns or attaches to a cold plan-fixer strand for the plan-conflict record `begin-batch` left, blocks until the strand ends, and reports its verdict.
- A new stencil for the plan fixer's prompt, rendered at call time from the stencils directory.
- The plan-fixer record in webster's `state.json`, its leftover-strand cleanup on run respawn and on reset, beside the cold verify-gate fixer's.
- Master's stencil: its plan-conflict rung runs `fix-plan` first and ends stuck only when the fixer cannot clear the conflict.
- Docs: `internal/websterengine/doc.go`, `internal/webstercli/doc.go`, `contracts/specs/webster-spec.md` where it lists the verbs or the conflict handling, `contracts/specs/llm-model-spec.md` where it says which strands each role runs, and `docs/overview.md` if it lists webster's verbs.
- Out of scope: the loom recipe, the shed Webster adapter and the plan-review instructions, which keep routing a conflict that still stands when Master ends stuck.

## Decisions

### D1: The plan fixer is a cold strand that `lyx webster fix-plan` spawns, never a Master fork

Master runs `lyx webster fix-plan` backgrounded after a `plan_conflict` refusal, exactly as it runs `fix-verify`.
The verb spawns one strand, role `plan-fixer`, under the webster segment, with a Go-rendered prompt, and blocks until it ends or the recovery timeout passes.
Rationale: a fork inherits Master's whole context, so it is neither small nor cheap, and Master's rules forbid it and its forks from editing the plan;
a cold strand starts with only the conflict record and the cards it names.

### D2: The plan fixer runs at the master role's model choice

The strand runs at `RoleMaster`'s resolved model choice, `sonnet[medium]` in the template, with `recovery_timeout_min` as its timeout.
No new `webster.yaml` key is added.
Rationale: the job is a mechanical edit of names in card files, which the lighter role fits, and a new model key would ripple through the config template, its classification and every role table for no knob anyone has asked for.

### D3: The fixer edits only the cards the findings name, then runs `lyx webster rebaseline --card NN` for them

The prompt names the conflict record, the plan directory and the reply path.
The fixer reads the record and each named card, edits each card's target names, paths or glyphs to match the current tree, and runs `lyx webster rebaseline` naming every card it edited.
It never edits a source file, the overview, a card no finding names, or anything outside the plan directory besides its reply file, and it never runs git or any other `lyx webster` verb.
When `rebaseline` refuses, the fixer runs `lyx webster restore-plan` so the plan on disk is the recorded one again, and reports the conflict as one for the plan writer.
Rationale: `rebaseline` already holds the line on which cards may change mid-run and fabric-commits the accepted plan, so the fixer needs no write rule of its own beyond the prompt.

### D4: The fixer judges whether a fix is mechanical, and its reply carries the verdict

The reply file's first line is `verdict: fixed` or `verdict: plan_writer`, followed by a short note per card.
`fixed` means every finding was answered by edits that leave what each card does unchanged: a renamed symbol, a moved path, a glyph respelled.
`plan_writer` means some finding needs a change to what a card does, such as dropping, adding or reshaping work, and then the fixer leaves the plan as recorded.
`fix-plan` returns `{"status": "done", "verdict": "..."}` on a reply with a valid verdict line, and a reply without one counts as `plan_writer`.
Rationale: only the agent reading the card and the tree can tell a rename from a design change; reading the verdict from its first line keeps the verb's decision mechanical.

### D5: Master's rung: `fixed` re-begins the batch, anything else ends stuck as today

On `{"plan_conflict": ...}` Master runs `fix-plan` backgrounded and ends its turn.
On its completion: verdict `fixed` → call `begin-batch NN` again and go on with the loop;
verdict `plan_writer`, a dead fixer or a `fix-plan` refusal → write `outcome: stuck` quoting the original `begin-batch` refusal, as the rung says today.
Master never runs `fix-plan` for `plan_drifted`, which stays a stuck rung.
The conflict record `begin-batch` wrote stays on disk until the next `begin-batch`, so a stuck run still reaches the shed adapter's reroute claim unchanged.
Rationale: the loom reroute keeps handling what the fixer cannot, so neither shed nor the recipe changes.

### D6: `fix-plan` is bounded per finding set and per batch

`fix-plan` refuses, spawning nothing, when no conflict record is on disk, when the record's findings equal those an earlier plan fixer of this run was given, or when this run's plan fixers for the record's batch already number two.
Each refusal names the way forward: write `outcome: stuck` quoting the `begin-batch` refusal.
A non-terminal last record is attached to, so a re-entrant call after a crash or a `running` return waits on the same strand.
Rationale: this edge lets a non-plan-writer agent change pending cards mid-run; it can skip a plan-review round only for a finding set a fixer has not already failed, and at most twice per batch, so a fixer that does not clear the conflict reaches the existing reroute after at most two extra `begin-batch` calls.

### D7: The plan fixer's session is not in the run audit's session list

The plan fixer's record is kept beside, not inside, the cold verify-gate fixer records, and `recordedSessions` does not list it.
In its place, `fix-plan` records the worktree's HEAD at the spawn, and once the fixer ends, a HEAD that moved or a worktree with uncommitted changes turns the verdict into `plan_writer`, with a warning naming the cause.
Rationale: its one legitimate write is to the plan directory, which the audit classes as a correctness violation for every listed session;
its plan edits are bounded by `rebaseline`'s acceptance rules and by `begin-batch`'s own re-resolution of the edited plan, and the HEAD and clean-tree check stops a code edit from passing as a plan fix.

## Constraints

- PATTERN-refusal-way-forward: every new `fix-plan` refusal names its way forward, and a test reaches it.
- PATTERN-stencil-ownership: the fixer prompt is read at call time through `internal/stencilstore`; it points at the plan spec rather than restating it.
- PATTERN-parent-directive: the fixer's stencil renders the parent and edit directives; its skills and color go on the launch spec.
- PATTERN-shuttle-provider-seam: the strand starts through the existing `Starter` and stops through the stop verb.
- PATTERN-friction-capture: `fix-plan` refusals go through the CLI's refusal-note wrapper like the other webster verbs.
- PATTERN-tolerant-records: the new `state.json` field loads absent as nil.
- PATTERN-test-speed and PATTERN-test-isolation: tests through `lyx gate test`, faked starter and reed, one `tierN` tag each.

## Auto-mode assumptions

- A `plan_defect` conflict, at a first `begin-batch` with no batch begun, also goes to the fixer: the fixer's verdict, not the cause, decides whether it is mechanical.
- The fixer's model is the master role's, per D2, so a hub that wants a different one changes `master` or waits for a follow-up key.

## Open risks

- A fixer that misjudges a design change as mechanical can land a card that does something other than the plan writer meant; `begin-batch`'s re-resolution only checks that the cards match the tree.
- A file the fixer wrote and git ignores escapes D7's HEAD and clean-tree check.

## Acceptance criteria

- `fix-plan` with a conflict record spawns one strand at the master role's model with role `plan-fixer`, records it in state, and on a reply whose first line is `verdict: fixed` returns status `done` with verdict `fixed`.
- A reply with `verdict: plan_writer`, or without a verdict line, returns verdict `plan_writer`.
- A `fixed` reply from a fixer that moved HEAD or left the worktree dirty returns verdict `plan_writer` with a warning.
- `fix-plan` refuses with a way forward when no conflict record exists, when the record's findings equal an earlier plan fixer's, and when the batch already had two plan fixers.
- A re-entrant `fix-plan` call attaches to a non-terminal plan fixer instead of spawning a second one.
- A leftover non-terminal plan fixer is stopped and marked dead on run respawn, and stopped on reset.
- The rendered fixer prompt carries the conflict record path, the plan directory, the reply path, the parent and edit directives, and the rebaseline and restore-plan instructions.
- Master's stencil's plan-conflict rung runs `fix-plan` before writing stuck, re-begins the batch on `fixed`, and keeps `plan_drifted` stuck.

## Notes for the plan writer

- `FixVerifySpawnOrAttach`, `FixVerifyAwait` and the `fix-verify` verb are the shape to follow; `ClaimConflictRoute`'s finding-line reader gives the record's findings.
