```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. For each change the diff makes, I traced what else it forces and whether the diff leaves any of that wrong.

**Per change: the affected set, and whether it is covered**

1. **`parent-directive-parent.md`: "then end your turn: your run waits for the answer…"**
   - It is rendered non-interactively by: `dobbyengine/tidy.go:59`, `frictionengine`, `gatefixer/prompt.go:77`, `landingshed/prsession.go`, `loomcli/driverprompt.go`, `mergeresolve`, `samplerengine`, `seatengine/prompt.go:66`, `shedadapters/bouncer.go`, `shedarbiter`, `sheddriver/tacklerprompt.go`, `weaverengine/render.go:95` and `websterengine/render.go`.
   - Roles run by the loom, burler, webster and batten drive runners have the probe: covered.
   - Roles run by the gate card fixer (`gatecli/cardfixer.go:128`) and weaver (`weavercli/wiring.go:44`) get no probe, and the grep finds no `SetReplyProbe` in either. The new claim is false for them: **NOT covered** (finding 1).
   - No doc or test quotes the old sentence: covered.
2. **`parent-directive-operator-ban.md` rewording:** no test or doc quotes the old text. The rule is broadened ("a question in your pane"), not weakened: covered.
3. **`driftengine/judge.go` `autonomousJudgeMode`:** `seats_test.go:199-200` asserts it through the constant, so it still holds. The judge stencil renders `{{.parent_directive}}` (`drift-template-judge.md:16`), so the mode text's "as your parent directive says" has a referent: covered.
4. **creel `awaits-reply` verb, `Request.From` and `Response.AwaitsReply`:**
   - The only sender is `GoClient.AwaitsReply`, so the doc's "only the Go client" claim holds.
   - The protocol constant's doc and creelengine `doc.go` are updated.
   - The `docs/overview.md` CLI verb lists (395, 913) list CLI verbs only, and the verb is not one.
   - `creelcli` `doc.go` is updated: covered.
5. **`Runner.SetReplyProbe` wiring:** all four `SetNotifier` callers get the probe. The probe-less runners are listed in the summary's Bounds: tender, weaver, card fixer, orchcli, `pairteardown`, `shuttlecli`, creel delivery, `looparm` and batten `wire.go:1431`. They are consistent with the summary but not with the directive (see 1).
6. **shuttle `Wait` hold:** shuttleengine `doc.go`, the `wait.go` package comment and `docs/overview.md:673` are updated. `PATTERN-daemon-wakeups` already lists shuttle's `Wait`, and the change adds no new wait: covered.
7. **PATTERN rules:**
   - agent-addressing holds, since the probe goes through creel.
   - test-only-code holds, since `AwaitsReplyProbe` is reachable from `main` through the wiring.
   - refusal-way-forward does not apply: it covers webster, shed and loom refusals, and the creel refusal names a way forward anyway.
   - Declared PATTERN changes: the directive declares none, so there is nothing to land.
8. **Commit messages:** the Root-Cause trailer is present on b75202db9. No Pattern-Departure or Loosened-Check trailer is needed.

**Findings**

1. **MEDIUM, consistency.** `contracts/stencils/parent/parent-directive-parent.md:11`
   - The new sentence "your run waits for the answer, which arrives as `[creel]` mail and starts your next turn" is rendered into every non-interactive role.
   - Gate card fixers (`internal/gatefixer/prompt.go:77`, runner at `internal/gatecli/cardfixer.go:128`) and weaver agents (`internal/weaverengine/render.go:95`, runner at `internal/weavercli/wiring.go:44`) run on runners with no `SetReplyProbe`.
   - For them the claim is false: an ungated card fixer that asks and ends its turn with its outputs present is still finalized before the reply, which is the defect the board entry names.
   - Fix: either wire the probe into those runners, or render the waiting clause only for roles whose runner holds the turn end.

Tool calls:
1. Grepped for the old and new stencil and judge-mode text, and for creel verb lists: needed every test and doc quoting the changed text.
2. Grepped judge-mode tests, the drift and seat directive wiring, and creel docs: needed the readers of the judge constant and the directive's renderers.
3. Listed `parentdirective` renderers and `NewRunner` constructors: needed every role that gets the changed directive and every runner that runs one.
4. Read the `looparm`, batten `wire.go`, tidy, gatefixer and weaver sites, and grepped for `SetReplyProbe`: needed to confirm which runners of directive-rendering roles lack the probe.
