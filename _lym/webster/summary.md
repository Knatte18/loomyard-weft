# Slack machine bridge: five batches built

All five batches reached `done`; the final head is `bf3353d0`.
No batch reported a `Pattern-Departure:` trailer, and no batch reported renames.
The scope guard's `deviations` lists are informational and are named per batch below.

- **01-machine-address-parser** (head `d3a9ab5a`): `internal/agentname` parses `<machine>::<name>` with the own-name refusal.
  Creel's journal carries and folds machine mail.
  Creel journals an orch's post to another machine and refuses every other edge.
  Deviations: `internal/creelengine/crossmachine.go`, `internal/creelengine/journal_test.go`.
- **04-machine-relay-verbs** (head `0643efca`): creel's program-only take, report and settle verbs, the machine mail hour, the deliver verb with dedupe and reply-to check, and the machine stamp.
  Deviations: six files under `internal/creelengine`.
- **06-creel-bridge-wiring** (head `6b89d7ab`): the Go client's machine verbs, the `slackBridge` seam, the envelope formatter and parser, the bridge channel name, the peers record, the bridge map and the new Slack calls.
  Deviations: eight files under `internal/creelcli` and `internal/slackengine`.
- **09-approve-and-revoke** (head `7c83dc09`): the operator's `approve` and `revoke`, the app id in `status`, and the relay making bridge channels and posting machine mail with holds and the bridge poll.
  `record-batch` first failed the batch: four card 10 targets (`BridgeSender.Send`, `BridgeSender.SettleAccepted`, `BridgePoll.Run`, `BridgePoll.Wake`) were built unexported.
  One `recover-batch` run finished it as `done`.
  Deviations: ten files under `internal/slackengine`.
- **11-bridge-receiving** (head `bf3353d0`): the relay reads bridge channels, delivers to creel and resyncs channels it lacks.
  Bridge channels take the new operator and are archived with their orch.
  Deviations: eleven files under `internal/slackengine`.

Every digest carried `warnings: null`.
