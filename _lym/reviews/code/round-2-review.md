---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: internal/slackengine/keeper_test.go:598
    summary: retire's retry of a failed dropForwarding for an orch whose channel already stands archived has no test
    fix: In the Retire subtests of TestChannelKeeper, add one subtest that seeds an unlisted beam's open entry and its button, makes the forwarding file unwritable after the archive stands (for example replace the file with a directory through the harness before the call), calls Retire(ctx, nil, false) and expects an error, then restores the file and calls Retire(ctx, nil, true) and expects nil, with the button dropped and no second Archive call. Fold it into the existing refusal-retry subtests if their harness already reaches it.
---

### [LOW] A failed button drop after the archive stood is retried in code but pinned by no test

**Location:** internal/slackengine/keeper.go:439 and internal/slackengine/keeper_test.go:598

**Issue:** `channelKeeper.retire` now keeps an archived entry's orch in its pass while the forwarding file still lists it.
When `dropForwarding` fails after the archive stood, the orch is marked refused and the next pass with `retryRefused` set runs the drop alone (`open` is false, so `retireEntry` is skipped).
The code does this correctly.
`AButtonOfAnUnlistedBeamIsDroppedWhetherItsChannelWasNeverMadeOrAlreadyArchived` covers the drop with no failure, and the archive-refusal subtests cover the refused-orch retry for a failed archive or rename.
No test fails the drop itself and then expects the retry to finish it, which is the behavior the focus names.

## Focus findings

Each focus question is settled below; none is a defect.

- `RunRelay` ends when the forwarding list is empty after a retire pass.
  `wakeLoop.wake` re-reads the forwarding file after the retire and sets `exiting`; `run` then returns nil.
  `RelayEndsWhenTheRetireDropsTheLastButton` reaches it through a hint-woken wake.
  An unreadable forwarding file does not end the relay.
  A relay started with no button cannot come from `EnsureRelay`, which answers `OutcomeForwardingOff`; it would end at its first wake.
  The tests that run a relay with a button off, such as the refused-hub test, exit earlier or set a button.
- `retire` drops the forwarding entries of retired beams that have no map entry or an archived one.
  The orch list takes the map's orchs plus the forwarded ones the map lacks, and skips only the prime and an orch that is neither open nor forwarded.
  The failed-drop retry is correct in code and is the subject of F1.
- `worktreeOrch` is reached for the prime, a listed beam, a pair that is no beam and an unlisted worktree through `hubforge` and a real beam pair, with no fake seam.
  `orchProject` is reached for the empty slug, a project entry, no entry and an unreadable `board.json` over a real board.
  The root handle `P0` is a named constant in `slackcli`, which is the route the parent's ruling in the decision record allows.
- The Slack relay bullet in `pattern/PATTERN-daemon-wakeups.md` names the inbox and forwarding-file watch, never within a second of the previous wake's start, with the on-disk fallback from one second to sixty.
  That is what `wakeLoop.watch`, `awaitFloor` and `run` do.
  No Pattern-Departure trailer is in the range, and no declared PATTERN change is missed by the diff.

## Rest of the range

The diff makes what the decision record asks:
- a per-orch `slack` button kept in the hub's forwarding file;
- no button for a pair's orch;
- a relay that runs only while a button is on;
- the `[slack presence]` notice and its role-file line removed;
- the machine-first, lower-cased channel name;
- old channels left alone;
- the turn-end hook posting only for an orch whose button is on.

Nothing outside the limits named in the record changed beyond the docs that describe the change.
