```yaml
lens: code-affected-set
outcome: reported
```

Task: code-affected-set lens. I list what each change forces outside its own lines, then the members it leaves wrong.

**Per change, the affected set and whether it is covered:**

- **`MachineBridge.RelayPostReady`, new interface method** (`crossmachine.go:29`). It has two implementers, `creelcli.slackBridge` and `creelfake.Bridge`, and both gain the method. No other type implements `SendReady`. Covered.
- **`creelengine.post` now asks the bridge before it stores a post to `<shortname>:SLACK`** (`verbs.go:193-213`).
  - In-process callers through `PostWith`:
    - the daemon's `dispatch` VerbPost (`server.go:313-321`), which carries ReceiverLive and Warning on both the plain path and the waiting path through `postStamp`. Covered.
    - the slack turn hook, `slackcli.replyHook` (`hook.go:173-194`). It posts to `<shortname>:SLACK` through `orchClient`, which is `creelcli.NewStrandClient` (`wiring.go:130-131`). That is a **strand** sender, so the new check now applies to it. Its own button check comes first, so in practice it is refused only when the bridge cannot read its files, or for a pair's orch. Either way it logs the reply as lost, which matches what the relay did before. The behaviour is acceptable, but the change description says otherwise (finding 1).
  - Program senders through `NewProgramClient` (`wiring.go:255`) are the relay itself and are exempt. Covered.
  - Docs:
    - `creelengine/doc.go` and `creelcli/doc.go` are updated. Covered.
    - The orch role file `contracts/orchrole/role-slack.md:7` still says mail posted to SLACK while the button is off "is dropped" (finding 2).
- **CLI `postCmd` Long text** (`cli.go:376-404`). It is within the `maxLongBytes` cap (`cmd/lym/clitree_test.go:169`).
  - The three removed facts are idle wake, the reply-command mark and the `--parent` resolved receiver.
  - The behaviour behind each still exists: `deliver.go:325`, `reminder.go:51`, and `cli.go` setting `fields["receiver"]`.
  - Each is still documented in `creelcli/doc.go` at lines 90, 22 and 72, as the cap test's message asks.
  - Covered.
- **`Response.ReceiverLive` and `Warning` doc comments** (`protocol.go:134-139`). No contract, spec or skill outside the two package docs mentions `receiver_live`. Covered.
- **`creelfake.Bridge.Refuse` now keys both methods, and its parameter is renamed.** Two existing callers:
  - `verbs_test.go:448` posts to machine addresses only.
  - `machinemail_test.go:417` sends a Reply to a machine.

  Neither reaches `RelayPostReady`, so there is no cross-effect. Covered.
- **PATTERN entries.**
  - `PATTERN-agent-addressing`'s Slack boundary says an agent reaches Slack only by creel mail to `<shortname>:SLACK`. The diff keeps that intact. Covered.
  - `PATTERN-refusal-way-forward` binds only webster, shed and loom, so it does not bind creel. The new refusals carry a way forward anyway.
  - No declared PATTERN changes. Covered.
- **Commit messages.** There are no `Pattern-Departure:` or `Loosened-Check:` trailers, and none is needed. The commit message of `69a4e4ca` says "Program posts and a daemon without the bridge are unchanged", which is accurate. Covered.

**Findings:**

1. MEDIUM, consistency, `_lym/landing/summary.md` ("What changes"):
   - The change description says "lym's own program senders, such as the dobby reminders and the slack turn hook, keep their current behaviour".
   - The slack turn hook is not a program sender. It posts as the orch strand, so the new bridge check now applies to it.
   - Evidence: `internal/slackcli/hook.go:173,191,208-216` (`orchClient` uses `newStrandClient`) and `internal/slackcli/wiring.go:130-131` (`creelcli.NewStrandClient`).
2. LOW, consistency, `contracts/orchrole/role-slack.md:7`:
   - The orch role file still tells the orch that mail it posts to `<shortname>:SLACK` while its button is off "is dropped".
   - The diff now refuses that post with a way forward, and adds a warning when the relay is down. The role text no longer describes what the orch sees.
   - Evidence: `contracts/orchrole/role-slack.md:7`, `internal/creelcli/slackbridge.go:116-118`.

Tool calls:
1. Searched for `SendReady(` implementers and for docs about posting to SLACK. The context had not swept every implementer or contract.
2. Read `contracts/orchrole/role-slack.md` and searched specs, stencils and skills for `:SLACK`. The role doc was outside the manifest.
3. Read `internal/slackcli/hook.go` and `wiring.go` client construction to check the sender kind of the slack turn hook, a caller one hop beyond the manifest.
4. Searched contracts, plugins and other package docs for `receiver_live`, to find further readers of the changed output.
