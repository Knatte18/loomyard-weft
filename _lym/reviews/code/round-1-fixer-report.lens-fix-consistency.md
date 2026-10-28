```yaml
lens: fix-consistency
outcome: reported
```

No findings blocking; two stale siblings found.

Task: fix-consistency lens over the F1–F5 change set.

**[MEDIUM] `internal/battenshed/doc.go:37`** — the "Limits" sentence says "a run whose pid is dead is not read".
- The F1 fix added line 29 to this doc, saying a running run whose waiting process is gone is read, with its activity only.
- `internal/shuttleengine/activity.go:112-114` shows `ReadAgentActivity` reads that run and sets `WaiterGone`; `battencli/wire.go:605-619` passes the reading on unfiltered.
- The same doc now says both "is read" and "is not read" for the same run.
- The fixer reworded neighbouring lines (21, 26, 28) but not this one.

**[LOW] `internal/shuttleengine/activity.go:46`** — `AgentActivity` is still documented as "the reading of one live run".
- The new `WaiterGone` field's doc, a few lines below, says such a reading "does not show the agent live".
- F1's fix key asked for the "live" wording to be revised in the doc comments of `liveRunStrands`, `agentActivityFor` and `battenshed/doc.go`; the type's own comment was left as it was.
- Related and unchanged: `activity.go:1-2` ("live shuttle runs") and `battencli/wire.go:590` ("reads the live agent runs").

**[LOW] `internal/battenshed/doc.go:26`** — "lasts while any live run of the child reads usage-limited".
- The fixer changed the "live run" wording to "read run" in lines 21 and 27 but missed this twin.
- A waiterless reading never carries a usage limit, so this is not a behaviour error; it is wording left inconsistent with its siblings.

The remaining checks held:
- F3: `typedInstruction`'s doc comment matches its code.
- F4: the `carryover.go` doc comment matches the new `promptLine` logic, and the `PromptLine` field doc in `rundir.go` is unchanged and still accurate.
- F1: `shuttleengine/doc.go:435` was updated to mention `WaiterGone`.
- F1: the `loomcli` doc comments were updated.
- F1: the three `activity_test.go` rows that read a waiterless run expect `WaiterGone: true`.

Tool calls made (1):
- Ran `grep` for "pid is dead", "live run" and "live agent run" across `internal/battenshed`, `internal/battencli/wire.go` and `internal/shuttleengine/activity.go` and `doc.go`, because my context lacked the unchanged sibling lines in those files.
