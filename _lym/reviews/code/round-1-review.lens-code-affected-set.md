```yaml
lens: code-affected-set
outcome: reported
```

Lens code-affected-set: every caller, doc, test and rule outside the diff's lines that the diff's changes force.

**Affected set per change**

- **`agentname.RoleRule` and `RoleFromText` (new).** Callers are `shuttlecli/agent.go` (Long text, `--name` flag help, `roleFromFlag`) and `open.go` through `roleFromFlag`. `open.go:34` writes the rule out as its own literal in the Long text instead of using `RoleRule`. Not covered (F2).
- **`open --role` now normalizes and refuses.** Covered:
  - Consumers are the board button's `run` line (`template_posix.yaml:71`, `template_windows.yaml:71`), whose ask text is updated and asserted at `config_test.go:120`.
  - `cli_test.go` drives `openCmd` with valid roles only, so no existing row changes meaning.
  - No other production caller passes `--role` and relies on it reaching reed unchanged.
- **New verb `shuttle agent`.** Two members:
  - `cli.go` package doc and `AddCommand`: covered.
  - `docs/overview.md` line 646 entry: covered.
  - `docs/overview.md:2044`, `` `lym shuttle run|open|manifest|state|await` ``: not covered (F3).
- **`Runner.StartSession` gets a new caller.** Its contract is in two places:
  - `run.go` doc: "The caller owns stopping it, through StopStrand".
  - `shuttleengine/doc.go` "Opened and unawaited sessions": "the caller stops it through StopStrand. The run record reads running until a stop ends it."
  - The agent verb never stops the session, records it nowhere and registers nothing. The existing callers (`landingshed/prsession.go:78`, `loomshed/samplersession.go:218`) each keep the strand GUID in a record, call `RegisterSession`, and stop through `StopStrand`. Not covered (F1).
- **`RunState.PID` and carry-over (`carryover.go`) as reached by the new caller.** The run records `os.Getpid()` (`run.go` launch, `PID: os.Getpid()`), the short-lived `lym shuttle agent` process. `unfinished()` and `runIsLive` then read the first agent's run as unfinished once that process exits. Not covered; reported in F1.
- **`agentconfig.RoleAgent` and `template.yaml` `agent:`.** Covered:
  - `config_test.go:77` iterates `Roles()` against the template.
  - `docs/overview.md:291` describes the module generically.
- **Stencil `operatoragent-template-start`.** Covered:
  - Registered in `entries` and in `roleOpeningStencils`.
  - `parentdirective_test.go:128,146` assert its parent and edit markers.
  - `roleskills_test` applies to it.
  - No doc lists stencils by name; `samplerengine/doc.go` names only its own.
- **`operatoragent.SessionSpec` producer.** Covered by the `shuttleparity` producer table row. `shuttlecli` is skipped by the literal scan; the literal sits in `operatoragent`, which the table lists.
- **Button templates (task, beam, prime agent and board rows).** Covered by `TestLoadConfig_TemplateDefaultsResolve`, and both OS templates changed alike. `buttonclick.go:157` refuses a comma in an ask text, and the new ask texts hold none.
- **PATTERN entries.**
  - `PATTERN-stencil-ownership` parent directive: rendered.
  - Role skills: on the spec, not asked of the agent.
  - `PATTERN-engine-parity`: producer row added.
  - `PATTERN-child-processes` "Run process owner": not covered (F1).
  - No declared PATTERN changes in the directive, so nothing to land.
- **Commit messages.** `eae3e6b` carries `Root-Cause:` naming the defect the diff removes: the answer passed unchanged, and a bare session. No `Pattern-Departure:` or `Loosened-Check:` trailers, and none needed.

**Findings**

- **F1.** MEDIUM, design, `internal/shuttlecli/agent.go` (the `StartSession` call). The new caller breaks `StartSession`'s documented contract that the caller owns the stop.
  - **What breaks:** no `StopStrand`, no stored GUID and no runproc registration, unlike the PR-Review and sampler callers.
  - **Consequence one:** the run record reads `running` forever, with `PID` naming the exited `lym shuttle agent` process.
  - **Consequence two:** carry-over reads it as unfinished. The next `lym shuttle agent` with the same role in that worktree, `agent` by default, appends the earlier session's operator-typed turns to its prompt as "Operator instructions carried over". It also sends the replacement notice, even while the first agent is still live.
  - **Consequence three:** the strand has no `DoneWhen` and is never Held, so reed's resume relaunches a closed agent.
  - **Evidence:** `internal/shuttleengine/run.go` StartSession doc ("The caller owns stopping it, through StopStrand") and launch `PID: os.Getpid()`; `internal/shuttleengine/doc.go:110-111`; `internal/shuttleengine/carryover.go:121-127,170-187`; `internal/reedengine/lifecycle.go:183-215`; `internal/landingshed/prsession.go:78-95`.
- **F2.** LOW, consistency, `internal/shuttlecli/open.go:34-35`. The `open` Long text restates the role rule as a literal instead of `agentname.RoleRule`, the named constant `agent.go` uses. That breaks `PATTERN-code-conventions` "Named constants", and the two can drift. Evidence: `open.go:34`; `agentname.go:120`.
- **F3.** LOW, consistency, `docs/overview.md:2044`. The verb list `lym shuttle run|open|manifest|state|await` leaves out the new `agent` verb, while the module entry at `docs/overview.md:646` was updated. Evidence: `docs/overview.md:2044`.

Tool calls:
1. Grep for ask-text copies, verb lists, role-list consumers and role docs. My context lacked the members outside the diff.
2. Read `shuttleengine/doc.go` "Opened and unawaited sessions", `overview.md` 2038-2048 and the stencil-name lists. They were needed for the StartSession contract and the stale verb list.
3. Grep for unknown-role handling in agentconfig and for copies of the rule literal. They were needed for the merge-door check and the named-constant check.
