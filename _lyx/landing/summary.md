# Webster fixes a mechanical plan conflict with a cold plan fixer

When `begin-batch` refused with `plan_conflict`, Master ended stuck at once, and under loom the whole run rerouted to Plan-Burler-Review: a new plan-writer round and its perch, even when the fix was two renamed test names in one card.
Now a plan conflict goes first to a small cold plan fixer inside the Webster run.
Only a conflict whose fix would change what a card does still ends Webster stuck, and still reroutes through the unchanged conflict-record claim in the shed adapter.

## What changed

- **New verb `lyx webster fix-plan`** (`internal/webstercli/fixplan.go`), modelled on `fix-verify`: Master runs it backgrounded after a `plan_conflict` refusal.
  It spawns, or attaches to, one strand with role `plan-fixer` at the master role's model, with `recovery_timeout_min` as its timeout, and blocks until the strand ends.
  Its envelope carries `status`, and `verdict` plus `reply` on done.
- **The fixer** (`contracts/stencils/webster/webster-template-plan-fix.md`) reads the conflict record and the named cards, edits only their stale target names, paths and glyphs, and runs `lyx webster rebaseline --card NN` for each card it edited.
  When `rebaseline` refuses, or a finding would change what a card does, it runs `lyx webster restore-plan` and replies `verdict: plan_writer`; otherwise it replies `verdict: fixed`.
- **Engine** (`internal/websterengine/planfixer.go`): `FixPlanSpawnOrAttach`, `FixPlanAwait` and `RecordPlanFixerTerminal`, a new `State.PlanFixers` record, and `ReadConflictRecord` to read back the record `begin-batch` writes.
  The bounded wait the cold verify-gate fixer used moved into `coldfixer.go`, and both fixers now share it.
- **Master's stencil**: the plan-conflict rung runs `fix-plan`, calls `begin-batch` again on `fixed`, and writes `outcome: stuck` quoting the original refusal on any other verdict, a dead fixer or a refusal.
  `plan_drifted` stays a stuck rung with no fixer.
  `lyx webster fix-plan` joins Master's awaited shell prefixes, so a turn end waiting on it is never an arrival.
- **Cleanup**: run entry stops a non-terminal plan fixer and saves it dead/died, and `reset --to start` stops every recorded plan-fixer strand, as for the verify-gate fixers.
- Docs: `internal/websterengine/doc.go` gains a "The plan fixer" section; `internal/webstercli/doc.go`, the `webster` help, `docs/overview.md` and `llm-model-spec.md` name the new verb.

## Design decisions

- **A cold strand, not a Master fork.** A fork inherits Master's whole context, and Master's rules bar it and its forks from editing the plan.
- **The master role's model, no new `webster.yaml` key.** The job is a mechanical card edit; a separate key can follow if a hub asks for one.
- **The fixer's verdict decides what is mechanical**, not the finding's check name, since `glyph-not-found` covers both a rename and a removed unit.
  A `plan_defect` cause goes to the fixer too.
- **Bounds:** `fix-plan` refuses findings an earlier plan fixer of the run already had, compared as a set, and a batch that already had two plan fixers.
  Each refusal's way forward is ending the run stuck, so a fixer that cannot clear a conflict hands it to the plan writer after at most two extra `begin-batch` calls.
- **Audit:** the plan fixer's session is not in the run's write evidence, since its one legitimate write, to the plan directory, is a correctness violation for every listed session.
  Instead a `fixed` verdict is overruled to `plan_writer`, with a warning, when HEAD moved during the fixer or a path that was clean at the spawn is left uncommitted.
  The decision record says "a worktree with uncommitted changes"; the check compares against the paths already uncommitted at the spawn, because a tree already holding uncommitted paths would otherwise overrule every fix.
  A file the fixer writes that git ignores escapes this check.
- **Unchanged:** the loom recipe, the shed Webster adapter and the plan-review instructions, which still reroute a conflict record left when Master ends stuck.

## Testing

- `internal/websterengine/planfixer_test.go` (tier0): every refusal and its way forward, the spawn's spec, prompt and record, the attach, each verdict reading, both overrule causes, a dead fixer and the terminal record.
- `internal/webstercli/fixplan_test.go` (tier1): the verb's spawn and attach envelopes for a fixed reply and a dead fixer, the saved record, and the no-record refusal.
- Existing rows extended for the plan-fixer strand cleanup at run entry and reset, and for the awaited shell prefixes.
- `lyx gate test --upto 1` on the computed scope is green.
