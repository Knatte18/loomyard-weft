---
round: 1
focus:
  - "internal/creelcli/cli.go `postCmd` Long text: the last commit removed help lines about idle-receiver wake, the reply-command mark and the --parent resolved receiver. Settle whether each removal still describes behavior the code has, and whether the board entry licenses dropping it."
  - "internal/creelcli/slackbridge.go `RelayPostReady`: settle whether every sender kind that reaches it is refused or accepted as the board entry asks. Cover an EXTERNAL sender, an orch thread name, a beam orch, and a sender whose `agentname.TaskSlug` or `Format` call fails."
  - "internal/creelengine/verbs.go `post` and `slackRelayCheck`: settle whether the bridge refusal leaves the store untouched, including the order against `interruptBeforePost` and the Wait and Reply paths. Settle whether the swallowed `FormatProgram` error and the receiver comparison can skip the check for a SLACK receiver spelled differently."
  - "internal/creelengine/verbs.go `post` end: settle whether `ReceiverLive` and `Warning` reach the Response and the stderr line on every path, including a waiting post whose Warning rides a progress response and a post stored after an interrupt."
  - "Scope against the board entry: it asks that a post to SLACK with no relay or a button off say so, by a refusal or by `delivered: false`. Settle whether the diff's mix of refusal for button states and `receiver_live` plus warning for a down relay covers both cases, and whether the refusals each name a way forward."
  - "internal/testkit/creelfake/bridge.go: `Refuse` now keys SendReady and RelayPostReady in one map and was renamed in its parameter. Settle whether existing tests that use `Refuse` for an orch still mean what they say, and whether the shared map causes a cross-effect."
  - "Test economy and coverage: `TestSlackBridge_RelayPostReady` and `TestStore_PostToTheSlackRelay` are the new tests. Settle whether each asserts the refusal wording, the no-store guarantee and the live and down answers, and whether any added behavior (a daemon without the bridge, a program sender) has no coverage."
  - "Doc comments in creelengine/doc.go, creelcli/doc.go, protocol.go and verbs.go: settle whether each states the new behavior accurately against the code, per the repository's own comment conventions."
---

Seed focus from the board entry, the diff and the commit message of `69a4e4ca`.
