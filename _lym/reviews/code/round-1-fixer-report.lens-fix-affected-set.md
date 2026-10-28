```yaml
lens: fix-affected-set
outcome: reported
```

Findings from the three set searches:

**F1 (MEDIUM, premise is a set of callers): the fix covers every member.**
- `ReadAgentActivity` has three production callers.
- `battencli/wire.go:605` reads waiterless runs on purpose.
- `loomcli/looparm.go:169` and `loomcli/decision.go:856` now skip them.

**LOW — `internal/shuttleengine/activity.go:46`, the `AgentActivity` type comment.**
- It still reads "the reading of one live run", but the type now carries waiterless runs through `WaiterGone`.
- The finding's own fix list named the "live" comments of `liveRunStrands`, `agentActivityFor` and `battenshed/doc.go`. This comment is a member of the same set that the fix did not reword.
- `internal/shuttleengine/doc.go:433` ("live or not") is already accurate.

**LOW — `internal/battencli/wire.go:590`, the `readAgentActivity` doc comment.**
- It says "reads the live agent runs", but this is the one caller that reads waiterless runs too.
- It is a stale "live" comment of the same kind the finding asked to fix.

**F3 (LOW, premise is a set of sites): no uncovered site.**
- `typedInstruction` is the only place that treats a prompt as a slash command.
- The other `HasPrefix(..., "/")` hits are path checks unrelated to prompts.
- `claudeengine/doc.go:108` still says "empty ... for a slash command". That stays correct, but it is silent on the new path rule. It is a stale-sibling candidate only, below the LOW bar.

**F4 (LOW): no uncovered site.**
- `operatorInstructions` has one caller, `carriedInstructions` at `carryover.go:142`, and it already reads the record's `PromptLine`.

**F2 and F5 (test-gap findings): left unchecked.** Neither premise is a set of callers or sites, and the changes added the requested rows directly.

Tool calls: one `grep` pass over `internal`, `docs` and `PATTERN.md`, covering all three set searches. It checked the callers, "live" wording, slash-command sites and `operatorInstructions` sites, which my context lacked outside the files already read.
