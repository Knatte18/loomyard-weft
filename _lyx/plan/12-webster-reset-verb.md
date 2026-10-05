# Card 12 — webster-reset-verb

**Edit:**
- `internal/webstercli#websterCLI.addVerbs`
- `internal/webstercli#Command`
- `internal/websterengine/doc.go`
- `contracts/specs/refusal-spec.md`

**Create:**
- `internal/webstercli/reset.go`
- `internal/webstercli/reset_test.go`

**Uses:**
- `internal/websterengine#PlanReset`
- `internal/websterengine#ResetDeps`
- `internal/websterengine#ResetPlan`
- `internal/websterengine#ResetTarget`
- `internal/fabricengine#Fabric.ResetPairWarp`
- `internal/fabricengine#NewMutations`
- `internal/fabricengine#MutationRecord`
- `internal/fabricengine#Fabric.CurrentBranch`
- `internal/webstercli#fabricSync`

**Intent:**
D6: `lyx webster reset --to start|pre-fix` performs in Go every branch reset a Webster refusal names, so no recovery needs the denied `git reset --hard`.

`reset.go` builds the `reset` subcommand in Webster's cobra subtree (PATTERN-cli-cobra), geometry resolved by the existing `cliwire` wiring (PATTERN-cliwire-sole-wiring); `addVerbs` registers it through `noteRefusals`, and the group's `Long` verb list gains its line.
`--to` is required and takes `start` or `pre-fix`; any other value refuses naming the two.
The verb, under the state-mutation lease:
loads state.json,
calls `PlanReset` with the pair's branch read through the fabric handle (`Fabric.CurrentBranch`) and the pair check (the weft checkout holds that branch's weft name),
calls `Fabric.ResetPairWarp` with a `NewMutations` recorder, the plan's SHA, the parent branch from the origin record and the plan's `OwnPaths`,
clears `State.PreFixHead` and saves,
then fabric-syncs state.json like `accept-audit`.
It changes no other Webster state; `run --fresh` or a plain `run` does the rest, as the way-forward texts order them.
A fabric refusal is surfaced as the verb's error with fabric's own reason.
In standalone mode (no fabric handle) the verb refuses: standalone has no pair to guard, way forward `git reset --keep <sha>` naming the resolved target, which keeps uncommitted changes and is not a denied command.
Naming `git reset --keep` here is the overview's departure from D7, awaiting operator confirmation.
On success the envelope carries `target`, `sha`, and the fixed mutation keys: `mutations`, always an array, holding the `worktree_reset` entry, and `partial`, always false (PATTERN-mutation-record); a refusal before the reset is a bare error envelope.
Bound: the verb can discard only commits above a run-recorded commit on the task's own branch, plus uncommitted tracked changes to paths the run itself wrote;
it cannot move another branch, take a raw SHA, touch the parent branch, the weft or untracked files, or override a refusal (no `--force`).

`refusal-spec.md` gains a row per new refusal in the webster section, each quoting its way forward: unknown `--to` value, run busy, no run, mid-merge, detached HEAD, parent branch, non-pair branch, no recorded target (start, pre-fix), missing target commit, target not an ancestor of HEAD, dirty tracked path the run did not write, and standalone mode (PATTERN-refusal-way-forward).

Tests (`reset_test.go`, `integration`, a hubforge hub with a task pair, PATTERN-hubforge-fixtures):
`--to start` on a pair branch with batch commits above the start commit and dirt on a path a fake run transcript wrote resets HEAD and the file, leaves an untracked file, and emits a `worktree_reset` mutation with `partial: false`;
`--to pre-fix` resets to the persisted pre-fix head after a rejected fixer commit and clears `preFixHead`;
`--to start` resolves the octopus merge-base when recorded starts diverge;
and one reaching test per new refusal row, asserting its way forward.
Document the verb, its bound and its refusals in `internal/websterengine/doc.go`.

**ImpactSummary:** one new verb registered beside the existing ones; refusal-spec gains its rows and no existing row changes.
