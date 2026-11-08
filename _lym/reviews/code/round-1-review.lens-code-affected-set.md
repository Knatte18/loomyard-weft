```yaml
lens: code-affected-set
outcome: reported
```

**Lens: code-affected-set.** For each change the diff makes, here is the set I found outside the touched lines and whether it is covered. I used 3 tool calls, so the checks are by grep, not by running anything.

**Per change**
- **Presence moved from per hub to per machine (`PresencePath`, `slack-presence.json`):** the readers are `hubgeom.SlackGeometry`, `orchengine` (`SlackPresencePath`, `WithPresence`), `orchcli.orchPaths`, `dobbyengine.WaitReminderDeps.Away`, the relay, `slackcli` and `EnsureRelayWhileAway`.
  - The code is covered, because every reader gets its path from `Geometry.PresencePath`.
  - Several doc comments still say "the hub's presence"; see F1.
- **`SetPresence` removed:** I found no non-test caller left.
- **`role-slack.md` trimmed:** `orch-role.md` and `docs/overview.md` were updated with it.
  - `role-slack.md:8` still says `[slack presence]` mail "tells you the new presence". The relays still send that notice, so this is accurate.
- **New `HookInput` fields (`Prompt`, `Reply`) and `HookEventPrompt`:** the callers are `creelcli/hookread.go` and `hookwait.go`, which only run on tool-boundary and Stop hooks.
  - A prompt event never reaches them, so they are covered.
  - The codex `HookInput` and the codex engine ignore `TurnHookCommand`, which `Spec` documents and the summary lists as a bound.
- **`Spec.TurnHookCommand` and `buildSettings` signature:** the only production caller is `claudeengine.go:224`, and the only setter is `orchcli/start.go:210`, so both are covered.
- **`agentname.FormatMachineAddress`:** it is used in `outbound.go` and documented in `agentname`'s package doc, so it is covered.
- **PATTERN and `pattern/` docs:** the diff changes no `PATTERN.md` or `pattern/` file, and I saw no declared changes in my directive, so nothing is owed there.
  - `PATTERN-agent-addressing`'s Slack boundary says an agent reaches Slack only by creel mail to that program. The Stop hook posts through the creel client as the orch, so it stays consistent.
  - `PATTERN-daemon-wakeups` line 89 describes the presence watch. The watch is now on the credential directory, but the wording still holds.
- **Production code imports `testing` (`panebin.go` `slackLineExecutable`):**
  - `grep` found no rule in `PATTERN.md` or `pattern/` for or against `testing.Testing()` in production code.
  - I raise nothing under the repository's rules.
- **reed line 3, `status-interval` and the new options:** no doc outside the code comments and `docs/overview.md` describes the bar, so there is nothing else to update.

**Findings**

- **F1 — LOW, consistency, 6 stale "hub's presence" comments.** Presence is now the machine's, but these still say "the hub's":
  - `internal/orchcli/paths.go:22`: "sets the hub's presence record".
  - `internal/orchengine/state.go:42`: `SlackPresencePath` is "the hub's presence record".
  - `internal/orchengine/prompt.go:9, 28, 31`: "while the hub's presence is slack".
  - `internal/orchengine/doc.go:52`: "the hub's presence is slack".
  - `internal/dobbyengine/waitreminder.go:28`: "Away reports whether the hub's presence says...".
  - Fix: reword each to "the machine's presence".

I found no other gaps in the affected set.

**Tool calls**
1. Grep for `SetPresence`, `presence.json` and "hub's presence" across the tree, plus the stencil and contract mentions of `[slack presence]`, `role-slack` and "while the presence is". My context did not show the readers of the old per-hub presence or the docs and stencils.
2. Grep for `TurnHookCommand`, `HookInput` and `HookEvent` callers, the `shuttleengine` file list, and Slack mentions in `PATTERN.md` and `pattern/`. I needed the other consumers of the changed hook types and whether a pattern entry names Slack.
3. Grep of `pattern/` and `PATTERN.md` for `testing.Testing`, the docs for bar and `status-interval` mentions, the diffstat of `PATTERN.md` and `pattern/`, and the stale-comment sites in `orchcli/paths.go` and `dobbyengine/waitreminder.go`. I needed to know whether a rule covers the production `testing` import, whether any PATTERN change was made, and the exact stale comment text.
