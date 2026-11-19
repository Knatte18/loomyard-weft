```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. I traced each change in the diff to the board entry's goal points, and each goal point to the changes that build it.

**Decisions** (the record's `## Decisions` is empty and it has no `Added after Discussion` entries, so the goal points come from the board entry under `## Goal`):
- G1: a post to SLACK while no relay runs says so. It either refuses with the state and `lym slack start`, or answers ok with `delivered: false` and the reason.
- G2: a post to SLACK while the sending orch's slack button is off says so, refusing with "press the button" or answering `delivered: false`.

**Per change:**
- C1: `verbs.go` `post`, `slackRelayCheck` and `relayDownWarning` (diff 002:156–220). Traces to G1 and G2.
- C2: `crossmachine.go` `MachineBridge.RelayPostReady`, with its interface doc (002:1–21). Forced by C1.
- C3: `slackbridge.go` `RelayPostReady` (001:62–121). The button-off refusal traces to G2 and the `RelayLive` answer to G1. The refusals of a non-orch strand, EXTERNAL and a pair's orch go past G2's literal wording ("the sending orch"). I count them as traced: the same goal asks that a post the relay drops silently says so, and those senders' mail is dropped by `outboundRoute` (`internal/slackengine/outbound.go`, the `!ok` branch). No finding.
- C4: `creelfake/bridge.go` `RelayPostReady`, `SetRelayLive` and the shared `Refuse` (003). Forced by C2.
- C5: `TestSlackBridge_RelayPostReady` and `TestStore_PostToTheSlackRelay`. Forced by C1 and C3.
- C6: doc comments in `creelengine/doc.go`, `creelcli/doc.go`, `protocol.go` and the `verbs.go` `PostResult` comment. Forced by C1 and C3.
- C7a: the `cli.go` Long text gains the SLACK `receiver_live` and refusal lines (001:33–35). Forced by C1 and C3.
- C7b: the `cli.go` Long text drops three lines: idle wake, the reply-command mark, and the `--parent` resolved receiver (001:9–25, commit `fe73fb3e`). This is forced by C7a plus the `maxLongBytes` cap of 2000 (`cmd/lym/clitree_test.go:169`). The Long block measured about 2011 bytes at base, 2293 at `d79b3f1a` and 1996 at HEAD. The dropped facts are still documented in `internal/creelcli/doc.go:22`, `:72` and `:90`. No scope finding.
- The comment re-wraps in `d79b3f1a` (test file headers) are forced edits.

**Per decision:**
- G1 is built by C1, C2, C3 (`RelayLive`), C6 and C7a. It answers `receiver_live: false` plus a warning, not the `delivered: false` the entry names (F1).
- G2 is built by C1, C2, C3 (the button-off refusal naming the status bar), C6 and C7a.

**Findings**

- F1 — LOW, class `scope`, `internal/creelengine/verbs.go:212-214` (`post`, `relayDownWarning`).
  - **Summary:** for a down relay, the diff reports through the existing `receiver_live: false` field and a stderr warning instead of the `delivered: false` field the entry names.
  - **Effect:** a caller reading the JSON envelope must know that `receiver_live` on `ly:SLACK` means "relay running", not "pane live". `_lym/landing/summary.md` records this as the writer's own design decision ("no new field was added"), not a recorded correction from the parent.
  - **Evidence:** board body under `## Idea` ("answer ok with a `delivered: false` field and the reason"); `verbs.go:212-214`; `cli.go` RunE `fields["receiver_live"]`; the summary's "Refuse vs. `delivered: false`" bullet.
  - **Fix:** add the `delivered` field the entry names, or have the parent record the substitution with `lym loom decision add`.

Tool calls:
1. `git show` of the `cli.go` post Long text at base, `d79b3f1a` and HEAD, piped through `wc -c`, plus a grep of `creelcli/doc.go` for idle wake. My context had no Long sizes, and I needed them to judge whether the help-text trim was forced by the cap and whether the dropped facts are still documented.
