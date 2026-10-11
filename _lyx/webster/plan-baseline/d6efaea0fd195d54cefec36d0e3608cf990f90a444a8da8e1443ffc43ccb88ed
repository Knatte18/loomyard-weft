# Card 7 — Quiet, api-error and driver-dead notices lead with the child's state

**Edit:**
- `internal/battenshed#conditionText`
- `internal/battenshed#innerRunProducer.judgeAgents`
- `internal/battenshed/doc.go`
- `internal/battenshed/notice_test.go`

**Uses:**
- `internal/battenshed#haltText`
- `internal/battenshed#noticeKey`

**Intent:**
Decision D6.
Commit `17b09e97f` moved the child's state into `haltText` for state-changed notices only, so the notices with their own text lost it.

`conditionText` renders a quiet, an api-error and a driver-dead condition as `haltText(status)`, then `; `, then the condition's own text, the shape the driver-busy notice already uses, for example `running at Plan-Write; agents idle for 25m0s`.
A state-changed condition keeps rendering as `haltText(status)` alone.
The driver-dead text becomes `driver strand is dead`, and in `judgeAgents` the two quiet texts drop ` while the child is running`, since the state now says it: `agents idle for <d>` and `no agent activity readable for <d>`.
The api-error text, `agent <producer> hit an API error: <text>`, is unchanged after the state.
Episode keys are unchanged: `noticeKey` holds the condition name, not its text, so a batten restart's marker written under the old wording still matches and the notice is not resent.

`doc.go` says every notice names the child's state and current row, for example `running at Plan-Write; agents idle for 25m0s`, beside its sentence on what a state-changed notice names.

Coverage extends existing tests in `notice_test.go`, with no new top-level test: `TestNotice_DeadDriverWhileRunning`, `TestNotice_QuietOnlyForRunningChildWithLiveDriver` and `TestNotice_APIErrorStall` assert each notice's condition starts with the child's state and current row followed by `; ` (for example `running at Plan-Write; ` for a fixture whose current producer is `Plan-Write`) and holds no `while the child is running`.
`TestNotice_RestartDoesNotRenotify` keeps covering the unchanged keys.
Grepping for the old texts finds only these `notice_test.go` assertions, which use substrings the new texts still hold.

**ImpactSummary:** the condition text of three notice kinds sent to the orch; keys, delivery, timing and the set of conditions are unchanged.

**Commit:** 7: batten: quiet, api-error and driver-dead notices name the child's state
