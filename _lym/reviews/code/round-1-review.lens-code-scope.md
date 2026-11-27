```yaml
lens: code-scope
outcome: reported
```

Task: the code-scope lens, which maps each change in the diff to a board-entry goal point and reports changes with no goal and goals with no change.

**The decisions.** The answer key is the decision record alone. The board entry's `## Goal` has three points, and the record has no `Added after Discussion` entries.
- **G1:** name the rule for the agent's name when asking for it. On a refusal, name the rule and ask again, or turn spaces into dashes.
- **G2:** start the agent with an opening stencil that says:
  - what it is;
  - which hub, worktree and task (or the prime) it sits in;
  - what it is for;
  - which skills it loads for that context, such as `ly:cli`, `ly:board` and the scribe skills.
- **G3:** give it the same instructions, guards and skills as any other lym session of its engine.

**Per change:**
- **C1** `internal/agentname`: `RoleFromText`, `RoleRule` and `TestRoleFromText`. Traces to G1, the conversion of spaces to dashes and the stated rule.
- **C2** `internal/shuttlecli`: `roleFromFlag`, used by `open --role` (open.go) and `agent --name` (agent.go). Traces to G1, the conversion and a refusal that names the rule.
- **C3** the `ask:` texts of the task, beam and prime agent buttons in `internal/reedengine/template_posix.yaml` and `template_windows.yaml`, plus the board button's ask text and the rows in `config_test.go`. Traces to G1, naming the rule when asking. The board button's change is in scope too: it asks for an agent name and runs through the same `open --role` conversion.
- **C4** the `operatoragent-template-start` stencil, its registration in `contracts/stencils/stencils.go` and its `roleOpeningStencils` row, plus `internal/operatoragent` (`SessionSpec`, `RenderPrompt`, `placeClause`) and its tests. Traces to G2: the stencil names what the agent is, its hub, its worktree, the board entry or prime, and what it is for.
- **C5** `internal/agentconfig`: `RoleAgent` and the `agent:` skill list `[scribe:prose, scribe:conversation, ly:cli, ly:board]`, followed by the language skills `SessionSpec` passes. Traces to G2's skills and G3. `PATTERN-stencil-ownership` says skills go on the launch spec and no stencil asks an agent to load one. Where that rule and the board entry's wording ("which skills to load" in the stencil) differ, the rule wins, so this is not a finding.
- **C6** the `lym shuttle agent` verb (`agent.go`), its wiring in `cli.go` (including the `reedGeometry` field), and the button `run:` lines in both templates plus `config_test.go`. Traces to G2 and G3: the button starts the agent through `Runner.StartSession`, the run-session path that delivers settings, shell guards, standing passages and skills.
- **C7** `internal/shuttleparity/producers_test.go`, the `buildOperatorAgent` row. Not built for a goal: C4 forces it, because `PATTERN-engine-parity` requires every new launch-spec producer to have a producer-table row. It also helps show G3.
- **C8** `docs/overview.md`, the module row and the shuttle verb text. Not built for a goal: C4 and C6 force it as a doc edit.
- **C9** commit ef3d2e625, which rewraps comments in `internal/operatoragent/spec.go`, `spec_test.go` and `internal/shuttlecli/agent.go`. Not built for a goal: C4 and C6 force it as a comment-format fix to their own files.

**Per decision:**
- G1: C1, C2, C3.
- G2: C4, C5, C6.
- G3: C5, C6, with C7 forced.

**Findings:** none. Every change traces to a goal point or is forced by one that does, every goal point has a change that builds it, and no commit in the range carries a `Pattern-Departure:` trailer that needs judging. G1's "ask again" branch is not built, but the goal gives it as one of two choices, so building the dash conversion satisfies the point.

Tool calls:
1. `git show --stat` of the three commits in the range. My context had the combined diff pieces but not which commit changed which file, which I needed to trace commit ef3d2e625.
