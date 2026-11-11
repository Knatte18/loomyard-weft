```yaml
round: 1
fixed: [F1, F2, F3, F4, F5, F6]
```

## Changes

- F1, `internal/creelengine/server_test.go`: added the row `WaitingPostAnswersReceiverLiveOnItsPendingAndFinalResponsesAndWarnsOnlyOfAReceiverThatIsNotLive` and the helper `openRawRequest`.
  **Fixed differently** from the proposal.
  The proposal asked to extend the existing row and assert `ReceiverLive` on the pending response.
  The client swallows pending responses, so that row cannot observe them; the new row reads the responses on a raw connection instead.
  It asserts `receiver_live` false on the pending response of a waiting post to the dormant strand, followed by the warning progress.
  It asserts `receiver_live` true on the pending and the final response of a waiting post to the live strand alpha, answered by a reply, with no progress response between them.
- F2, `internal/creelcli/cli_test.go`: the `receiver_live` subtest runs `post --wait-reply --timeout 5s` to the dormant strand and asserts exit 4 and stderr equal to the warning line, so exactly once.
  The post to a program receiver without `receiver_live` is not asserted: the rig has no cheap program receiver, and the omit branch is a nil check on the response.
- F3, `internal/creelcli/doc.go`: took the reviewer's documentation alternative.
  The doc says a `--wait-reply` resumed after a dropped connection answers no `receiver_live`.
  Keeping the value from the pending response would need a client change, because the client consumes pending responses without handing them out.
- F4, `internal/creelengine/client.go`: the `CallWatching` doc says "waiting inject or post".
  The comment-length gate flagged that line at 120 characters, so the doc now breaks after "arrival order" and gives the nil-progress rule its own line.
- F5, `internal/creelcli/doc.go`: the doc says the warning line is printed before the answer line.
- F6, `internal/creelcli/cli.go`: the post help says the answer to a post to a strand carries `receiver_live` and a receiver that is not live gets a `creel: warning:` line.
  The verify run then showed `TestCLIForm_EveryCommand` pins a cap of 2000 bytes on a command's Long text (`maxLongBytes` in `cmd/lym/clitree_test.go`), which the help exceeded at 2134 bytes.
  The sentence is now shorter, and trimmed wording elsewhere in the help (the interrupt paragraph, the expect-reply paragraph, one example and one word) brings the help under the cap.
  The earlier statement that no test pins a cap was wrong.

Commits: one per finding, `f0e01fb4` (F1), `30007881` (F2), `a20fce8e` (F3, F5), `9bc482c7` (F4), `8bb6b604` (F6), and a follow-up for the self-check changes.

## Self-check

- fix-code-correctness, LOW: the `doc.go` line on a resumed `--wait-reply` ran to 120 characters. Fixed as proposed, by shortening it.
- fix-proposal, LOW: the F6 help sentence claimed `receiver_live` for every post. Fixed as proposed: it now says the answer to a post to a strand.
- fix-proposal, LOW: the F3 doc line claimed behaviour no test pins ("prints the warning no more than once"). Fixed with my own change: the claim is dropped and the line keeps only the absent `receiver_live`.
- fix-consistency: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
