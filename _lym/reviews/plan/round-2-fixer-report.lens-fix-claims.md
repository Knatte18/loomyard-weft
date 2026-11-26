```yaml
lens: fix-claims
outcome: reported
```

Lens: fix-claims. I checked every sentence the fix added or rewrote in cards 8 and 9 against the code at HEAD.

no findings

What each claim rests on:
- **Card 9, dobby bullet.** The re-exec runs `syscall.Exec` on `os.Args` (`internal/selfexec/selfexec.go:92`), so it re-enters `lym beam dobby`. That verb calls `runDobbyWith(..., hubDobbyDeps, ...)` (`internal/beamcli/cli.go:715`), so every start and every re-exec builds the deps through `hubDobbyDeps`. `fixerTimeout` reads `tapestry.implementer` through `configreg.SettingLookup` (`internal/battencli/wire.go:807-811`).
- **Card 9, shed test bullet.** `step` is not in `verbUsesLightweightWiring` (`internal/loomcli/cli.go:338`), so `armAt` runs the full `c.wire` (`internal/loomcli/arm.go:271`). `loomPreStepUnmarked` returns `KindBusy` at the run-lock probe (`arm.go:627-641`), before `seedAndCommitBootstrap` (`:645`) and `reed.Up()` (`:658`).
- **Card 8, rollback text.** `boardCriticalSection` saves the store, then runs `afterSave`, then renders, all under a lock that `defer` releases (`internal/boardengine/board.go:103-107`, `:182-196`). A rollback that runs before the function returns therefore runs under the lock.
- **Card 8, render-failure test row.** The render writes with `os.WriteFile(fullPath, ...)` (`internal/boardengine/render.go:44`). A directory at that path makes it fail with EISDIR, after the save.
- **Card 8, tier0 row.** `carryHub` builds its fixture in a temp dir with no git (`internal/hubreconcile/carry_test.go:19-33`), so the row can stay in tier0.

Tool calls:
1. Read `ArmAt`, `fixerTimeout` and the callers of `hubDobbyDeps`, to check the arm route and the dobby's tapestry read; my context lacked them.
2. Grepped the callers of `hubDobbyDeps` and how `armAt` wires, because the first grep found no call with parentheses and I had not seen the wiring branch.
3. Read `verbUsesLightweightWiring`, the render's write call and the `carryHub` fixture, to check the full-wire claim, the render-failure mechanism and the tier0 claim; none were in context.
4. Grepped the exec call in `selfexec`, to confirm that a re-exec reuses the same argv and so goes back through `hubDobbyDeps`.
