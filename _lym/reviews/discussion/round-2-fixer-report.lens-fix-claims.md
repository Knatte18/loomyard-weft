```yaml
lens: fix-claims
outcome: reported
```

I checked whether the claims the fix added or rewrote hold against the code at HEAD.

- MEDIUM — D4, the Drift-Baseline bullet sentence "the agent may record a scope from the board entry alone, which then stands" (and the same words in the third Open risks bullet). The agent has no route to the board entry. Under `subject: decision-record`, `subjectFields.reading()` in `internal/driftengine/seats.go` gives the agent only the record's path. Only `SubjectBoardEntry` names the slug and `lym board get`. The baseline stencil (`contracts/stencils/drift/drift-template-baseline.md:20`) reads the subject only through `{{.subject_reading}}`, and nothing else in it points at the board. An agent pointed at a missing record has nothing to read, so the likely outcome is a Stuck or a scope recorded from nothing. This claim came from the reviewer's proposed fix.

- LOW — D13, the added sentence "never added to `retiredLoomKeys` or darn's Migrate drop list". Darn has no Migrate drop list. `internal/darnengine/config.go:186` `MigrateConfig` only moves the retired `tree_scope_verify` key and drops no list of keys. A reader or implementer will look for a list that does not exist. Name darn's `MigrateConfig` instead. This claim also came from the reviewer's proposed fix.

The other added claims hold:
- `missingSections` today matches lines whatever fence they sit in (`validate.go:113-129`).
- `decisionWayDarnWriter` is used by three refusals, and `decisionWayEntryLost` by the post-append refusal for a committed record lacking the new entry (`decision.go:383-395`, `711-714`).
- `PrepareRunSettings` runs `moveRunSettings`, then `Migrate`, then `checkLocalDocument`.
- On a second pass, a passed capture takes no new capture and an existing scope record stands (`driftengine/doc.go`). Today a darn run's scope agent reads the board entry through `SubjectBoardEntry`.
- `goto` resets no drift state: only `await.go:91` writes `AwaitedAt`, and no goto path touches drift.

Tool calls:
1. Grepped `internal/darnengine/config.go` for Migrate and its drop logic, to check the claim about darn's Migrate drop list.
2. Grepped the drift baseline stencil for any slug or board pointer and listed the files with goto functions; my context had neither.
3. Read the baseline stencil's markers and `seats.go` lines 70-95, to see what the record subject tells the agent.
4. Grepped the goto files for drift state; nothing matched, and no `gotoentry.go` exists.
5. Grepped for what writes or clears `awaited_at`, to confirm the drift state survives a goto.
