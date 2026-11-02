```yaml
lens: fix-claims
outcome: reported
```

Task: check that each claim the fix added or rewrote holds against the code at HEAD.

**LOW** — card 3 (`_lym/plan/03-webster-fixer-on-core.md`), the rewritten `testReplyPath` bullet.
- **Changed text:** "becomes the placeholder reply path the cold fixer's `gatefixer.RenderPrompt` inputs carry".
- **Summary:** The new doc comment would describe the constant too narrowly, because the plan fixer's prompt test also passes it.
- **Evidence:** `internal/websterengine/template_test.go:976` calls `websterengine.RenderPlanFixPrompt("/lym/webster/plan-conflict.md", testReplyPath, ...)`. The constant stays shared by the cold verify fixer's render and the plan fixer's render. A comment that names only the `gatefixer.RenderPrompt` inputs leaves out a live use.

Every other added or rewritten claim holds:
- **Card 4, `render.go` lines:** lines 1, 4, 25, 32 and 35 hold the asset list, the `RenderVerifyFixColdPrompt` caller sentence, the cold-prefix composition, the friction list and the pattern list.
- **Card 4, `recoverbatch.go`:** `RecoverAwait` calls `awaitTerminal` (`recoverbatch.go:610`), which calls `TurnEndedAfter` and `StrandLive` (`:826`, `:830`).
- **Card 10, `GotoGuard` comment:** its current wording is at `internal/shedengine/goto.go:19`.
- **Card 10, in-effect case:** `AdmitRelaunch` returns `inEffect` (`internal/sheddriver/relaunch.go:17`).
- **Card 10, guard order:** `Relaunch` runs `checkGuard` before `req.Admit` (`goto.go:232-236`).

Tool calls:
- One grep over `recoverbatch.go`, `sheddriver` and the `websterengine` tests. My context did not show which function in `recoverbatch.go` calls `TurnEndedAfter` and `StrandLive`, where `AdmitRelaunch` is defined, or the other uses of `testReplyPath`.
