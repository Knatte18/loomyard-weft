```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. Trace each diff change to the board entry's goal points and check that every goal point is built.

**Decisions.** This is a decision record alone, with no `Added after Discussion` entries.
- **G1:** an agent that asks with `--expect-reply` waits for the reply before it ends.
- **G2:** every run agent, the drift judge included, asks its parent by creel mail rather than in its pane.

**Per change:**
- **C1, creel `awaits-reply` verb** (`protocol.go`, `server.go` dispatch, `verbs.go` `Store.AwaitsReply`, `creelengine/doc.go`, `goclient.go` `AwaitsReply` and `AwaitsReplyProbe`, `creelcli/doc.go`): traces to G1, as the signal the hold needs.
- **C2, shuttle reply hold** (`run.go` `SetReplyProbe` and fields, `replyhold.go`, `wait.go` Wait branches and `turnStartSeen`, `shuttleengine/doc.go`): traces to G1.
- **C3, runner wiring** (`battencli/drive.go`, `burlercli/wiring.go`, `loomcli/wiring.go`, `webstercli/wiring.go`): traces to G1.
- **C4, parent-directive line** "your run waits for the answer… starts your next turn" (`parent-directive-parent.md`): traces to G1, a wording forced by C2.
- **C5, operator-ban line** "a question in your pane is never the way forward, since nothing written there reaches your parent" (`parent-directive-operator-ban.md`): traces to G2.
- **C6, drift judge autonomous mode text** (`driftengine/judge.go`): traces to G2. The judge stencil renders `{{.parent_directive}}` (`contracts/stencils/drift/drift-template-judge.md:16`), so "as your parent directive says" has a referent.
- **C7, `docs/overview.md`** (creelcli line and the Wait hold paragraph): forced by C1 and C2.
- **C8, tests** (`creelcli/cli_test.go` subtest, `creelengine/awaitsreply_test.go`, `shuttleengine/replyhold_test.go`): forced by C1 and C2.
- **C9, commit f68024cf1**, comment rewrapping: forced, since it touches only comments the range adds.

**Per decision:**
- **G1:** built by C1, C2, C3 and C4, but only for the four wired runners (see the finding below).
- **G2:** built by C5 and C6. No other run stencil still sends a question to the pane outside interactive mode. `loom-template-discussion.md:124` is interactive-only, and `loom-template-plan.md:26` and `loom-template-parts.md:13` forbid asking the operator.

No change lacks a decision: no unrequested scope.

**Findings**

1. **MEDIUM, class `scope`.** Location: `internal/gatecli/cardfixer.go:128`, and less certainly `internal/weavercli/wiring.go:44`.
   - **Summary:** G1 is built only for runners that already had a notifier. The gate card fixer still has no reply probe, yet its prompt renders the changed parent directive, which now promises "your run waits for the answer, which arrives as `[creel]` mail and starts your next turn".
   - **Consequence:** a card fixer that asks with `--expect-reply` and ends its turn can still be finalized before the reply. That is the exact defect the entry reports, now under a stencil promise that is false for that role.
   - **Evidence:**
     - `internal/gatefixer/prompt.go:77` renders the non-interactive directive through `parentdirective.Directive(in.StencilsDir, in.ParentName, false)`.
     - `internal/gatecli/cardfixer.go:128-130` builds the runner with `SetPause` and `SetSessionEnd` only, with no `SetReplyProbe`.
     - The summary's Bounds section admits the exclusion ("Runners that have no notifier (… the gate card fixer …) get no probe"). That is the writer's choice, which no `Added after Discussion` entry records.
     - Weaver's writer also renders the directive (`internal/weaverengine/render.go:95`), and `weavercli/wiring.go:44` builds a runner with no probe. I did not confirm that this runner drives the weaver agent's Wait rather than only reset deps.
   - **Fix:** wire `creelcli.AwaitsReplyProbe` into every runner whose agent's prompt renders the non-interactive parent directive, starting with `gatecli/cardfixer.go`. Alternatively, defer the narrowed scope to the parent as a recorded decision.

Tool calls:
1. grep of the drift stencils for `parent_directive` and of all stencils for pane-question wording: needed to confirm the judge gets the directive and no stencil still sends questions to the pane, which my context lacked.
2. grep of the matching lines in the loom, landing, seat and engine stencils: needed to judge each hit's mode.
3. grep of `shuttleengine.NewRunner(` callers and parent-directive renderers: needed to find which runners lack the probe, beyond the four `SetNotifier` sites in my context.
4. Read of `looparm.go`, the `battencli/wire.go` revival runner, `gatecli/cardfixer.go`, `weavercli/wiring.go` and the gatefixer/weaver `parentdirective` calls: needed to tell which unwired runners drive agents told the run waits.
