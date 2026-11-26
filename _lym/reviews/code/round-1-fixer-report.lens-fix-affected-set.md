```yaml
lens: fix-affected-set
outcome: reported
```

Task: fix-affected-set check, rerunning the set-shaped findings' searches over the committed fix.

Findings left unchecked: F6, F8, F11, F12 (their members are the single branches or rows the review named). F4, F5 and F9 I checked from context only, with no new search.

1. **MEDIUM** (F3), `internal/shedadapters/bouncer.go` `judgeCall`: a re-attached judge whose wait errors still degrades to `Stuck` and spends bounce budget.
   - `attachAndWait` returns `result, true, err` from `run.Wait()` (`internal/shuttleengine/attach.go:89-91`).
   - `judgeCall` sends any `attachErr` to `degrade` before the harvest and before the new `runErr` branch. So an attached judge that gave no verdict still counts as a bounce, and one whose verdict is already on disk is not settled there.
   - The rewritten doc says "A run error without such a judgment fails the call the same way", and the `_lym/landing/summary.md` line says "A judge run that returned an error with no verdict fails the call". Neither holds on this path.
   - The probe-failure case (`attached` false) is pinned by `TestBouncer_JudgeCall_AttachErrorDegradesWithoutSpawning`, so a change has to split on `attached`, or the doc and summary have to limit the claim to a spawned run.

No other gaps:
- **F1:** every production builder of `gatefixer.Deps` that reaches `Await` now sets `ShuttleCfg` and `AnchorRoot`: webster's two waits, `loomcli/gatefixer.go`, `loomcli/wiring.go` and `gatecli/cardfixer.go`.
- **F2, F7, F10:** I found no stale statement of the wait order, of the final closure never spawning, or of the sweep that the change set contradicts.

Outside scope: `internal/gatecli/detachfixer.go` runs its own fixer wait. I did not check how it treats an interrupted fixer.

Tool calls (5):
1. Grep for `Await`/`Deps` builders, plus a span of `gatecli/cardfixer.go`: my context lacked the full caller list.
2. Grep for wait-order, Final and sweep wording: it returned nothing, so it was rerun in call 4.
3. Grep for judge-degrade statements and the `Attach` signatures: my context lacked how an attached run's wait error returns.
4. `attach.go` lines 84-100 and a docs grep: my context lacked the body of `attachAndWait` and the sweep and order wording elsewhere.
5. A span of `loomcli/wiring.go`: my context lacked the fields of its `Deps`.

No files changed and no commits.
