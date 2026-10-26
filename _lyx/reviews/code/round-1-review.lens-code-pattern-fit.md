```yaml
lens: code-pattern-fit
outcome: reported
```

Task: check the diff against the repository's rules for the code-pattern-fit lens.

**Rule list (held or broken)**

- `PATTERN-refusal-way-forward`, held for the new refusals. Every `fix-plan` refusal carries a `way forward:` clause: no record, equal findings, batch limit, no model-spec, engine cannot audit, the dead-fixer message, and the no-run refusal. `internal/websterengine/planfixer_test.go` reaches the five engine refusals. `internal/webstercli/fixplan_test.go` reaches the no-record refusal and the dead-fixer message. Broken in one narrow spot, see finding 1.
- `PATTERN-refusal-way-forward`, "No denied recovery", held. Neither `webster-template-plan-fix.md` nor the rewritten master rung names a denied form. The plan fixer's `git` use is a "never".
- `PATTERN-stencil-ownership`, held. `RenderPlanFixPrompt` reads through `stencilstore.Read` at call time, the stencil is registered in `contracts/stencils/stencils.go`, and it points at `loom/loom-plan-spec.md` instead of restating it.
- `PATTERN-parent-directive`, held. The stencil renders `parent_directive` and `edit_directive`, skills come from `roleSkills` on the spec, and color comes from `Segment: segmentcolor.Webster`. Neither the stencil nor the render asks the agent to load a skill or set a color.
- `PATTERN-shuttle-provider-seam`, held. The strand starts through `Starter.Start` and stops through `StrandStopper.StopStrand`, and the model comes from the resolved `RoleMaster` choice.
- `PATTERN-tolerant-records`, held. `PlanFixers` is `omitempty` and loads absent as nil.
- `PATTERN-friction-capture`, held. `refusalnote.go` records every verb's output at one tee, so `fixplan.go` needs no wrapper of its own.
- `PATTERN-agent-name`, held. The role is the `planFixerStrandRole` constant.
- `PATTERN-gitrepo-client-boundary` and `PATTERN-cwd-resolution`, held. Git goes through the `Geometry.git()` seam (`HeadSHA`, `DirtyPaths`), and every path comes from the told geometry.
- `PATTERN-fabric-git`, held. The verb only calls the existing `fabricSync`.
- `PATTERN-sole-parsers`, held. The conflict record is not in the listed parser set.
- `PATTERN-shell-guard-table`, `PATTERN-run-process-owner` and `PATTERN-spawn-observability`, held. The fixer is spawned through the same `Starter` seam as the cold verify fixer, with no new process start.
- `PATTERN-test-speed` and `PATTERN-test-isolation`, held. The engine tests carry `tier0` and the CLI test carries `tier1`, with faked starter, reed and git.
- CLAUDE.md "Docs land in the same commit", held. The package `doc.go` files, `llm-model-spec.md` and `docs/overview.md` are updated. `webster-spec.md` is untouched, but it mentions neither `recover-batch` nor plan-conflict handling, so nothing there needed changing.
- CLAUDE.md "Markdown: semantic line breaks", held in the stencils and docs I read.
- Declared PATTERN changes: the constraints section declares none, so nothing to check.
- Commit message and departures: one commit and no `Pattern-Departure:` trailer, so nothing to judge.
- Go comment convention, held for the new doc comments, apart from findings 2 and 3.

**Findings**

1. LOW, `design`, `internal/webstercli/restoreplan.go:19` against `contracts/stencils/webster/webster-template-plan-fix.md`
   - `restore-plan` is annotated `AudienceOperator`, yet the plan-fix stencil tells the spawned fixer to run it in two places: after a refused `rebaseline`, and when a finding is for the plan writer.
   - `rebaseline` is `AudienceRole`. I did not trace how a role session's index or settings treat an operator-audience verb.
   - If a role session cannot see or run it, the stencil's only recovery step strands the fixer, which is the stranded-agent case `PATTERN-refusal-way-forward` forbids.
   - The fix is to annotate `restore-plan` as role-callable, or to confirm that the audience does not gate execution.

2. LOW, `consistency`, `internal/websterengine/conflictrecord.go:67-80` (`ReadConflictRecord`)
   - It re-parses the record's `- ` finding lines inline, though `readFindingLines` in the same file already does that.
   - The decision record's note to the plan writer pointed at that reader. Two parsers of one record format can drift apart.
   - The `fix-plan` bound compares `ReadConflictRecord`'s findings against stored ones, while `ClaimConflictRoute` compares `readFindingLines`'s.

3. NIT, `consistency`, `internal/webstercli/batchpause.go:1` and `internal/webstercli/wiring.go:344`
   - The comments still list `begin-batch, recover-batch and fix-verify` and `run, recover-batch and fix-verify`.
   - `fix-plan` now makes the same up-front covered-pause refusal (`refuseCovered("fix-plan", ...)`) and spawns an agent.
   - Both comments are now incomplete.

4. LOW, `consistency`, `internal/websterengine/planfixer.go:~120-135` (`FixPlanSpawnOrAttach`)
   - Three error returns carry no way forward and no test reaches them: the reply-path `filepath.Abs` failure, the record-path `filepath.Abs` failure, and the stale-reply `os.Remove` failure.
   - These are operational refusals through the verb, which `PATTERN-refusal-way-forward` says should name one. The same applies to `RenderPlanFixPrompt` errors and the unparsable `spawnedAt` in `awaitColdFixer`, which are shared with `fix-verify`.
   - Likely covered by the spec's exemptions for transient internal errors; I did not check them.

**Tool calls**
1. A grep and sed of `webster-spec.md`, the CLI audience annotations and `refusalnote.go`. My context lacked the spec's verb coverage, the audience of `rebaseline` and `restore-plan`, and the refusal-note wiring.
2. A grep of `batchpause.go` and `wiring.go` for `fix-verify` mentions, to see whether the verb lists there were stale.
3. A grep of `AudienceOperator` uses and a read of `restoreplan.go`, to see what the operator audience gates.
