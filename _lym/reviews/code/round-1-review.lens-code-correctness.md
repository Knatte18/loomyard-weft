```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I traced each function and branch the diff adds or changes through concrete inputs.

**Per function or branch: the case I ran, and the verdict**

- `agentname.RoleFromText`:
  - `"My Agent"` → `my-agent`. Correct.
  - `"  board \t tidy  "` → `board-tidy`. Correct.
  - `""` or `"   "` → `""`, which `ValidateRole` refuses. Correct.
  - `"my_agent"` → kept, then refused. Correct.
  - `"--name"` (a leading dash) → kept, then refused. Correct.
- `roleFromFlag`:
  - `"My Agent"` → `my-agent`, nil. Correct.
  - `"my_agent"` → `--role "my_agent" is no valid agent name: use …`. Correct; the rule leads the first output line the client shows (`buttonclick.go:193`).
- `open` with `--role ""`: still refused as `--role is required` before normalization. Correct.
- `open` with `--role "   "`: `role != ""`, so it reaches `roleFromFlag`, which refuses it with the rule. Correct.
- `agent` with no `--name`: defaults to `agent`. Correct.
- `agent` with `--name ""` given explicitly: refused by `roleFromFlag`. Correct.
- `agent` with no `--model`: `choice` is the zero value, and `EngineName` gives the set's default engine. `PromptEngine` and the launch both use that engine. Correct.
- `agent` with a bad `--model`: refused as `--model: …`. Correct.
- `agent` when `languageconfig.Load` fails: the error goes out through `output.Err`. Correct.
- `agent` when `SessionSpec` fails because `NameShortname` is empty (the hub records none, `geometry.go:51`): the button fails with `operatoragent: HubShortname must not be empty`. That message names no way forward. It is an edge case, not filed.
- `agent` when `StartSession` fails: the error is reported. `launch` cleans up its run directory or strand (`run.go:177-202`). Correct.
- `agent` under a pause covering the slug: `waitWhileCovered` (`run.go:101`) blocks with no bound. The button's `runButtonScript` waits with no timeout (`buttonclick.go:281`). Its notices go to stderr, which the client never shows on success. The button hangs silently; `open` refuses instead. Finding F3.
- `agent`, a second start in the same worktree with the same name while the first session is alive. Finding F1.
- `agent`, the operator ends the session. Finding F2.
- `SessionSpec` with all inputs set: the fields match the test. Correct.
- `SessionSpec` with each required input empty: refused by name. Correct.
- `RenderPrompt` with `ParentName == ""` (prime) renders the none variant. `"ly:orch"` renders the parent variant with no operator ban. Correct as code; the wording is not my lens.
- `RenderPrompt` with no PATTERN overview: `pattern_directive` is optional and filled empty. Correct.
- `RenderPrompt` with a stencil missing: the error propagates. Correct.
- `placeClause("")` gives the prime clause, and `"fix-the-thing"` gives the entry clause. A beam worktree has a slug from `l.WorktreeName` (`hubgeom/buttons.go:39`) and gets the entry clause. That is plausible and not filed.
- `cli.go`: `reedGeometry` is set in PersistentPreRunE before any verb runs. Correct.
- Template rows: `{answer}` is filled through `sh.Quote` (`buttons.go:167`), so `My Agent` reaches `--name` as one word. Correct.

**Findings**

**F1 — BLOCKING, design.** Location: `internal/shuttlecli/agent.go:93` (`c.runner.StartSession(spec)` with `Role: in.Name` and no NameOverride). A second `lym shuttle agent` with the same name in the same worktree pastes every instruction the operator typed into the earlier agent session into the new agent's prompt. That includes the default name `agent` on a second click.
- `launch` records `PID: os.Getpid()` (`run.go` state, near line 771). That is the pid of the short-lived `lym shuttle agent` process, which exits as soon as the start returns.
- `runIsLive` (`activity.go:179-187`) therefore reads the first session's run as not live while its session is still running.
- `unfinished` (`carryover.go:121-129`) then reads it as true.
- `start` (`run.go:106-111`) appends `carryOverBlock(carried)` to the new prompt: "The session this one replaces ended before it finished … Follow them as the operator's own".
- `carried` comes from the predecessor's operator-typed turn starts (`carryover.go:82-90`), and every turn after the opening prompt in an interactive agent is one of those.
- `noticeCarriedInstructions` also logs a Warn and, when a notifier is set, sends a "replaces an unfinished session" notice. Nothing was replaced; reed numbers the new strand `agent-2` beside the live one.

**F2 — MEDIUM, design.** Location: `internal/shuttlecli/agent.go:93`. An agent session the operator ends leaves a dead pane in its own window, and its run record reads running for good. Every reed resume brings the session back.
- `strandAddSpec` passes `DoneWhen: nil` for an unawaited spec, because `spec.OutputFiles` is empty (`run.go` launch, `doneWhen = spec.OutputFiles`). The launch line carries no self-remove.
- Under reed's `remain-on-exit` (`strand.go:711`) the exited pane stays as a corpse.
- `planResumeLaunches` (`lifecycle.go:192-216`) relaunches every non-live, non-held strand whose done-when list is empty. Each closed agent therefore resumes on the next reed resume.
- Nothing calls `StopStrand` for this session; only `StopStrand` marks it Held.
- The `open` path it replaces touched a marker, self-removed and exited (`open.go:327-337`), so its window closed and a resume dropped it.
- `lym shuttle state` keeps listing the session as running.

**F3 — LOW, design.** Location: `internal/shuttlecli/agent.go:93`. Under a pause that covers the task's slug, the agent button blocks with no bound and shows nothing; `open` refused at once with a way forward.
- Evidence: `run.go:101` and `run.go:135-149`, where the waiter's notices go to stderr; `buttonclick.go:274-281`, which waits with no timeout and shows output only on a non-zero exit; `open.go:294-299`, the old refusal.
- The verb's `Long` text says nothing about the wait.

**Tool calls**

1. I grepped `operatorInstruction` and read `runTurnStarts` in `internal/shuttleengine/carryover.go`. My context showed that `carriedInstructions` calls them but not how a typed turn is classed as an operator instruction, which F1 needs.
