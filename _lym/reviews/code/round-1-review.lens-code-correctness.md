```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I ran each changed branch of the diff on one concrete input.

The diff changes no Go code. It changes only the prose of two stencils, so the "branches" below are the changed instruction rules, each run on one task shape.

Per-branch list:

1. **`drift-template-baseline.md` Step 1, new invariant paragraph.**
   - Input: a task saying "every calculated result stays unchanged" whose design rounds the Reynolds number.
   - Result: the agent expects no drift and names the contradiction as a risk. It does not resolve it. This is correct.
   - Boundary input: a task with no stated invariant. The paragraph does not trigger, and Step 2 and Step 4 behave as before. This is correct.
2. **`drift-template-baseline.md` Step 2, `full` bullet.**
   - Input: a task that changes behavior on purpose, with drift in one test and a second line moving in the same test.
   - Result: the verify agent judges the drifted test against the task and the expectation. The second line still shows as a regression, because that sentence is kept. This is correct, and no rule is dropped.
3. **`drift-template-baseline.md` Step 4, report rules.**
   - Input: an intended-drift task.
     - Per-test naming of expected drift and the "every other line must stay as it is" sentence are both retained.
     - The agent is barred from writing an acceptance reason or a tolerance such as a relative 1e-3 bound.
     - Result: correct.
   - Input: an invariant task.
     - The agent writes that it expects no drift and names each contradiction as a risk.
     - Result: this agrees with Step 1.
   - Input: an empty case, a task with no drift expected and no invariant.
     - Nothing in the text forces a drift list.
     - Result: correct.
   - Authority split: "the verify agent and the run's parent" matches `internal/driftengine/doc.go:64-65` and `internal/driftengine/doc.go:119-128`. Under scope `full`, the agent's acceptances are unruled and the parent rules them. This is correct.
4. **`drift-template-verify.md` Step 2, two added lines.**
   - Input: a "pure refactor" task with drift, and a baseline report that names the rounding design as a risk.
   - Result: the drift is a defect, and the risk naming does not excuse it. This is correct.
   - Input: a task that intends drift.
     - The neighbouring rule "Drift the task intends is acceptable" is unchanged.
     - The new sentence applies only to tasks stating an output-unchanged invariant.
     - Result: no conflict.
   - Input: drift from a parent commit merged after the baseline commit.
     - The sentence "Drift traced to a commit merged from the parent branch after the baseline's commit is acceptable too" follows the new lines and is untouched.
     - Result: parent-merge drift stays acceptable under an invariant, and the text does not contradict this.
5. **Rendering.**
   - Both stencils keep their substitution markers, and the diff touches no marker line.
   - This is consistent with the marker lists at `contracts/stencils/markers_test.go:66-67`.
   - Result: correct.
   - `autonomousVerifyJudging` (`internal/driftengine/seats.go:57`) still says "giving the reason the drift is intended".
     - It sits below the new verify lines and agrees with them, because a reason can only be given for intended drift.
     - Result: no contradiction.

**Findings:** none. I found no wrong result, contradiction between the two stencils, or inconsistent state from the changed rules.

Tool calls: none. The inherited context already held the diff, both stencils, `internal/driftengine/seats.go`, `internal/driftengine/doc.go` and `contracts/stencils/markers_test.go`.
