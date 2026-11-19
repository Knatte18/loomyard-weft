```yaml
lens: code-scope
outcome: reported
```

I'm reviewing the diff's trace to the board entry's goal points under the code-scope lens.

**Decisions.** The record has no `## Decisions` content and no `Added after Discussion` entries. The goal points come from the board entry's `## Idea`:
- **G1:** a post to SLACK while no relay runs says so. Either it is refused with the state and a way forward (`lym slack start`), or it answers ok with `delivered: false` and the reason.
- **G2:** a post to SLACK while the sending orch's slack button is off says so. Either it is refused with the state and a way forward (press the button), or it answers ok with `delivered: false` and the reason.
- **G3:** the sender is told, so it reports the result in its pane instead.

**Per change:**
- **C1, the engine seam and post check:** `MachineBridge.RelayPostReady`, `Store.slackRelayCheck`, `relayDownWarning`, and the `post` wiring that sets `ReceiverLive` and `Warning`. Traces to G1, G2 and G3.
- **C2, the bridge implementation:** `slackBridge.RelayPostReady`, with the button-off refusal and `RelayLive`. Traces to G1 and G2.
  - It also refuses a strand that is no orch, EXTERNAL, and an orch with no button. That part goes beyond G1 and G2 (finding 1).
- **C3, help text for SLACK in `postCmd` Long:** forced by C1 and C2.
  - Commit `fe73fb3e` removed three lines: idle wake, the reply-command mark, and the `--parent` resolved receiver. The cap test forced those removals (`cmd/lym/clitree_test.go:280`, whose message says to move the long page into the package doc).
  - All three facts still hold and stay in the package doc: `internal/creelcli/doc.go:22`, `:68-72` and `:90`. So no fact was lost.
- **C4, doc comments:** `creelengine/doc.go`, `creelcli/doc.go`, `protocol.go` `ReceiverLive`/`Warning`, `verbs.go` `PostResult`, and the `crossmachine.go` interface doc. All forced by C1 and C2.
- **C5, tests and fake:** `TestStore_PostToTheSlackRelay`, `TestSlackBridge_RelayPostReady`, and `creelfake.Bridge` gaining `RelayPostReady`/`SetRelayLive` with `Refuse` widened. Forced by C1 and C2.
- **C6, commit `d79b3f1a`:** comment re-wrapping in the files above. Forced by C4 and C5.

**Per decision:**
- **G1:** built by C1 and C2. The post is stored with `receiver_live: false` and a warning naming `lym slack start`. It does not use the `delivered: false` form (finding 2).
- **G2:** built by C2 and C1. The refusal names "press the button on its status bar".
- **G3:** built by C1, C2 and C3. Each refusal and the warning tell the sender.

**Findings**

1. **LOW, scope.** `internal/creelcli/slackbridge.go` `RelayPostReady` (refusal branches, around lines 96–118).
   - **Summary:** the bridge refuses three sender kinds the board entry does not name: a strand that is no orch, EXTERNAL (for example the operator's terminal), and an orch with no slack button (a pair's).
   - The entry asks only about "no relay runs" and "the sending orch has its slack button off".
   - Before this diff, these posts were stored and then dropped by the relay (`internal/slackengine/outbound.go` `postMessages`/`outboundRoute`, around lines 196–232). Now they are refused.
   - This changes behaviour for these senders, and no `Added after Discussion` entry records it. The change summary's own design decision is the writer's claim, not a license.
   - It falls under the same "sender not told" problem the entry describes, so the rating is LOW. The parent should confirm or refuse it with `lym loom decision add`; the code needs no revert.
   - **Evidence:** `internal/creelcli/slackbridge.go` (diff hunk at line 88 onward); `_lym/discussion/decision-record.md` `## Idea`.

2. **LOW, scope.** `internal/creelengine/verbs.go` `post`/`relayDownWarning` (around lines 212–244) and `internal/creelcli/cli.go` post print path (around lines 451–473).
   - **Summary:** for a relay that is down, the entry offers either a refusal or an ok answer with a `delivered: false` field and the reason. The diff does neither literally.
   - It reuses `receiver_live: false` in the JSON answer and prints the reason only as a `creel: warning:` line on stderr. The JSON answer carries no reason and no `delivered` field.
   - The intent, telling the sender, is met through the existing channel. The deviation is the field name and where the reason appears, and no added entry records it.
   - **Evidence:** `internal/creelcli/cli.go:451-473`, where `warn` writes to stderr and `fields` holds only `id`, `sender`, `receiver_live`; `_lym/discussion/decision-record.md` `## Idea`.

Tool calls:
1. Grepped `internal/creelcli/doc.go` for the help facts the last commit dropped, and ran `git show fe73fb3e` on `cli.go`. The context lacked the package doc's text, which I needed to tell whether the removals lost facts or were forced edits.
