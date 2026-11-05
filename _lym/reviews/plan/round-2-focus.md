---
round: 2
focus:
  - "Card 9 (`_lym/plan/09-slackengine-relay.md`) and card 4's `RelayState` (`_lym/plan/04-slackengine-files.md`): settle whether the whileAway respawn loop's restart cap, kept in `RelayState` fields `StartFailures` and `NextStartAt`, survives the relay's own state rewrite at its step 3, and whether a relay that dies before that step still counts a failure, as `PATTERN-child-processes` requires of a retry loop around a real spawn. Check that every struct field card 9 changes is a target of a card that creates or edits it."
  - "Card 9 `AwaitFirstRead` and card 10 `lym slack away` (`_lym/plan/10-slack-cli.md`): settle whether `AwaitFirstRead` can act on a `RelayState` or `FirstReadRefusal` written by a relay that ran before the one `EnsureRelay` just started, and whether a test row reaches that case."
  - "Card 10 `slack` group `PersistentPreRunE` and the hidden `relay` verb: settle where `relay`, spawned as `slack relay --hub-path <hub>` with the hub directory as working directory, gets the prime location that `creelcli.NewProgramClient`, `hubgeom.DobbyStart`, the pause geometry and `hubgeom.SlackGeometry` need, given `lymcwd.Resolve` refuses a directory that is no worktree root. Compare creel's `daemon --hub-path` verb in `internal/creelcli/cli.go`."
  - "Card 10 hidden `relay` verb with `--fork`, and card 9 step 4: settle whether the card's `**Intent:**` states what a by-hand `relay --fork` run lets through, from any pane and off Linux, where `internal/creelengine/identity.go` resolves a sender from the environment before any pane walk, and what bounds it."
  - "Cards 11, 12 and 13 (`_lym/plan/11-orch-relay-presence.md`, `_lym/plan/12-dobby-relay-tick.md`, `_lym/plan/13-anchor-relay-restart.md`): for each package `doc.go`, help text and `docs/overview.md` line a card lists under **Uses:**, settle whether the card's change makes the text it describes false, so it belongs under **Edit:**."
  - "Card 3 (`_lym/plan/03-creel-seams.md`) `internal/creelengine/doc.go` under **Extends:**: settle whether the card rewrites or contradicts any sentence already there, which `Extends` forbids."
---

Round 2 re-reviews the plan after the round 1 fixes.
Each entry names a place and the question to settle; none pre-states an answer.
