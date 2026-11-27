```yaml
lens: fix-affected-set
outcome: reported
```

Task: fix-affected-set check, whether the fixes cover every member of each finding's set.

**Unchecked findings:** none of the set-shaped findings. F1 and F4 were rerun by search; F2, F6, F7, F8, F9, F10 and F11 were checked from context. F3 is not the fixer's.

**Findings**

1. **MEDIUM, F4: `contracts/stencils/parent/parent-directive-parent.md:33`, not changed by this fix.**
   - It still reads "before sending a run back to a Webster row after a fold, record each folded call again with `lym loom decision add`".
   - A send-back from Drift-Recheck to Webster is inside the closed span, where the verb refuses, so the step cannot be carried out.
   - Lines 36-37 of the same list ("record every call before the run's first Code-Review round") do not say what to do with folded calls in that case.

2. **LOW, F1/F4: pattern-headroom way forward.**
   - The sites are `internal/loomshed/patternplan.go:28` and `contracts/specs/loom-plan-checks-spec.md:247-248`.
   - Both still send the reader to `decision add … --amends-plan` with no closed-scope route.
   - The fix's new closed-scope ways forward (`webster.go:38`, `wiring.go` reroute cap) send such a run to `Plan-Burler-Review`, where the plan gate can raise this check. The first remedy, shortening the prescribed lines, still works.

3. **LOW, F1: `internal/discussionparser/patternchanges.go:195`.**
   - The over-cap finding on an appended bullet names only "ask the parent to re-record the entry … through `lym loom decision add`".
   - A PATTERN-change add is refused in a closed scope; `onlyLiftsRedFirst` excludes it. It matters only if an over-cap appended bullet is first judged after the line.

4. **LOW, F1: `internal/boardengine/recordupsert.go:113-115`.**
   - For an entry a run holds, the only way forward is `lym loom decision replace <slug>`, which refuses once that run's scope is closed.
   - The board-note route is not named.

5. **LOW, F4: `contracts/stencils/loom/loom-template-arbiter.md:68-70` and `:77`.**
   - The override says "at Code-Review you rule in scope or hand on instead".
   - A Webster stop after a Drift-Recheck send-back is also closed but is not "at Code-Review", so line 77 still tells the arbiter to record an `--amends-plan` decision there, which refuses.
   - The fix edited this bullet and left the wording.

The other Go and stencil sites the searches returned are either covered, admitted by the new red-first exemption (`redfirstgate.go`, `gatecli/redfirst.go`), or reached only while the scope is open.

Outside my lens: the `TestInClosedScope` comment and field in `internal/loomrecipe/scopeline_test.go` still say "started Code-Review round" / `roundStarted` after the parameter was renamed to `reviewRan`.

**Tool calls (3 of 6)**
- Grep of non-test Go for decision-verb and parent-records wording: context held only the first fix-time search, filtered to a few packages.
- `git grep` of `contracts` and `plugins` for the same wording: context held a narrower search of `orchrole` and `stencils`.
- Spans of `parent-directive-parent.md`, `loom-plan-checks-spec.md`, `orch-role.md`, `board-policy/SKILL.md`, `patternchanges.go` and `recordupsert.go`: the candidate sites were not in context.
