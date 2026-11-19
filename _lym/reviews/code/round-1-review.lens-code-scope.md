```yaml
lens: code-scope
outcome: reported
```

Task: the code-scope lens. It maps the diff's changes to the board entry's goal points and lists the scope findings.

**Decisions (goal points of the board entry under `## Goal`; the record has no `Added after Discussion` entries)**
- **G1:** a post to SLACK while no relay runs says so. It either refuses with the state and a way forward (`lym slack start`) or answers ok with `delivered: false` and the reason.
- **G2:** a post to SLACK while the sending orch's slack button is off says so. It either refuses with a way forward (press the button) or answers ok with `delivered: false` and the reason.
- **G3:** the sender then reports the result in its pane. This is the purpose behind G1 and G2, and both carry it.

**Per change**
- **C1, engine check before store.** `MachineBridge.RelayPostReady`, `Store.slackRelayCheck`, `relayDownWarning` and the wiring in `post` (`internal/creelengine/crossmachine.go`, `internal/creelengine/verbs.go`). Traces to G1 and G2.
- **C2, the bridge's sender and relay judgment.** `slackBridge.RelayPostReady` in `internal/creelcli/slackbridge.go`:
  - The button-off refusal traces to G2.
  - `slackengine.RelayLive` for the live/down answer traces to G1.
  - The refusals of a non-orch strand, of EXTERNAL and of an orch with no button (a pair's) trace to no named goal point. They extend G2's intent to other senders the relay drops; see finding 1.
- **C3, fake bridge.** `RelayPostReady`, `SetRelayLive` and the shared `Refuse` in `internal/testkit/creelfake/bridge.go`. Forced by C1's interface change.
- **C4, tests.** `TestStore_PostToTheSlackRelay` and `TestSlackBridge_RelayPostReady`, plus the reworded file headers. Forced by C1 and C2.
- **C5, doc comments.** `creelengine/doc.go`, `protocol.go`, the `PostResult` doc in `verbs.go`, and `creelcli/doc.go`. Forced by C1 and C2, since they make those docs stale.
- **C6, `postCmd` Long in `internal/creelcli/cli.go`.**
  - The new SLACK sentence is forced by C1 and C2.
  - Commit `fe73fb3e` removed three lines: the idle-receiver wake, the reply-command mark and the `--parent` resolved receiver.
  - That removal is forced: the new text pushed Long past `maxLongBytes` = 2000 (`cmd/lym/clitree_test.go:169`).
  - All three facts are still true of the code and still documented in the package doc: `internal/creelcli/doc.go:22`, `:68-72` and `:90`. The cap's own message directs long text there (`clitree_test.go:281`). Not unrequested scope.

**Per decision**
- **G1:** C1 and C2 build it. They answer ok with `receiver_live: false` and a warning naming `lym slack start`, not a refusal. See finding 2 for the field form.
- **G2:** C1 and C2 build it. The post is refused, naming "press the button on its status bar".
- **G3:** C1 and C2 serve it. The refusal texts name "report the text in your pane".

**Findings**

1. **LOW, scope.**
   - **Location:** `internal/creelcli/slackbridge.go:96-111` (`RelayPostReady`).
   - **Summary:** the bridge also refuses a strand that is no orch, EXTERNAL, and an orch with no slack button. The board entry names only two cases: no relay running, and the sending orch's button off.
   - **Why LOW:** these refusals turn a stored post into a refused one for senders the entry never named. They do match the entry's purpose, since the relay drops these senders' mail without posting it.
   - **Evidence:** the relay drops such mail at `internal/slackengine/outbound.go:197-200` and `:217-231`. The entry's text is at `_lym/discussion/decision-record.md:12-13`.

2. **LOW, scope.**
   - **Location:** `internal/creelengine/verbs.go:212-214` (`post`), `internal/creelengine/verbs.go:234-240` (`relayDownWarning`), `internal/creelcli/cli.go:466-473`.
   - **Summary:** when the relay is down, the entry asks for an ok answer with a `delivered: false` field and the reason. The diff instead answers `receiver_live: false`, and the reason appears only on the stderr `creel: warning:` line. The CLI's JSON envelope carries no reason field.
   - **Weighing:** the summary at `_lym/landing/summary.md` justifies this, since the mail waits and is posted once the relay restarts. But the entry's field and reason-in-answer form is not what lands, and no `Added after Discussion` entry or parent decision records the substitution.
   - **Evidence:** `_lym/discussion/decision-record.md:13`; `internal/creelcli/cli.go:455` writes the warning to stderr; `internal/creelcli/cli.go:466-473` builds the envelope fields with no warning.

No unrequested-scope finding for the C6 help-text removals: they are forced by the size cap, and the facts survive in the package doc.

No tool calls were made: everything this lens needed was already in context.
