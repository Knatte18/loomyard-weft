---
verdict: APPROVED
prior:
  - id: F1
    status: resolved
    answer: Cards 1, 2, 4, 6, 7, 8 and 10 each add the allowlist entries for the functions they create with no caller, and the first-calling card removes each, as the guard's `RequireNoStale` demands. Card 5 and card 12 create no uncalled production function and need no entry. The fixer's variant is correct. `GoClient.DeliverMachine` is first called by card 11's `Mailbox.DeliverMachine` call, so card 11 removes it. Card 10's `programMailbox.DeliverMachine` and `bridgePoll.wake` are unreached until card 11, so card 10 adds them. Card 2's `machineOutboundEntry.message` is first called by card 3's `settledOutcomeLocked`, so card 3 removes it. Each card that adds or removes an entry lists `cmd/lym#testOnlyCodeAllowed` under `**Edit:**` and states the entries in its Intent. The keys match the guard's `<directory>.<Type>.<Method>` form.
  - id: F2
    status: resolved
    answer: Card 4 creates `ErrDaemonRefusal` and `daemonRefusal.Is`, so `IsDaemonRefusal` is `errors.Is(err, ErrDaemonRefusal)` and a fake outside the package can wrap the sentinel. Card 10 states that `fakeMailbox` scripts a refusal that way, and card 11's rows and Uses carry it. The existing `daemonRefusal` in `internal/creelengine/client.go` keeps its `Unwrap`, so the coded refusals still match their own sentinels.
  - id: F3
    status: resolved
    answer: Card 11's `operator` step now states that a failed `runPeerCommand` holds the cursor, keeps the retry under the reader's own failing key `<channel id> <ts>`, and after `retryWindow` answers through `PostThreadReply` or the channel and moves the cursor. It names the owning goroutines. Card 9 states that `runPeerCommand` reads only the `inbound` fields set at build and writes none, and the reader answers through its own `SlackClient`. That matches the `inbound` struct in `internal/slackengine/inbound.go`, where `failingKey`, `episodes` and `refusedOrchs` are touched only by `retryOrGiveUp`, `settle` and `answer`, none of which the reader calls. A `TestBridgeReceive` row reaches the failure.
  - id: F4
    status: resolved
    answer: Card 5's Intent now has a `Bound:` paragraph for the program-only deliver verb, stating the Linux pane and no-pane acceptance, the any-process case off Linux, and that it is creel's existing program-claim bound.
  - id: F5
    status: resolved
    answer: Card 10 lists `internal/slackengine/doc.go` and `pattern/PATTERN-daemon-wakeups.md` under `**Edit:**`, has no `**Extends:**` label, and names the doc.go line and the bullet's `and` it rewrites.
  - id: F6
    status: resolved
    answer: Card 9 compares the command's machine with the current and every former name case-insensitively in `handle`, before `runPeerCommand` and so before `CheckName`, and a `TestInboundRead` row with former name `Old_Box` and `approve old_box` pins it.
  - id: F7
    status: resolved
    answer: Card 11 has one row for a deliver whose reply-to creel refuses, skipped with one notice quoting the reason, and the unreachable receiver-refusal row is gone.
  - id: F8
    status: resolved
    answer: Card 5 drops `releaseGoneHolder` from `**Edit:**` and says why. In `internal/creelengine/sweep.go`, `isStrandName` is `agentname.Parse` succeeding, and `<machine>::<name>` does not parse as an agent name (D1), so unread machine-inbound mail with no forwarder takes the `eventUndelivered` branch already. The `TestSweep_RemovedHolders` row pins it.
  - id: F9
    status: resolved
    answer: Card 6 lists `internal/creelcli#trustField` under `**Edit:**` and rewords its comment. `trustField` exists in `internal/creelcli/cli.go` with the comment the card replaces.
---

No new finding.
The round is incremental, so a new finding needs a blocking defect on a changed hunk, and the changed cards 1 to 11 hold none.
The focus questions settle as follows:

- Production functions without a first caller: every one has an allowlist entry added by its creating card and removed by its first-calling card, so no batch boundary leaves a function unreached or a stale entry.
- The fake mailbox of cards 10 and 11 reaches the refusal `IsDaemonRefusal` matches, through `ErrDaemonRefusal`.
- The bridge-channel `approve` or `revoke` failure has a stated outcome and an owner for each piece of state.
- The program-only deliver verb states what it lets through and what bounds it, as the take, report and settle verbs of card 4 and the post of card 3 already did.
