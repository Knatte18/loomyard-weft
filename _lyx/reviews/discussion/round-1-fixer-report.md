# Fixer report — discussion round 1

## Changes

- F1 (fixed as proposed): D9 appends and syncs a sending event before shuttle's send, and the fold treats a sending inject with no outcome as unconfirmed; D6 lists sending; D2's binary-identity exit starts no new send and waits for in-flight sends to return and be journaled before closing the listener; a new Acceptance criterion covers a daemon killed mid-send.
- F2 (fixed differently): D4 now decides the precedence, but not by the review's example rule of refusing every in-pane claim.
  `lyx batten run` starts from the orch's pane, so that rule would make `ly:BATTEN` unreachable on Linux, which is the problem the review itself names.
  Instead, a program-sender claim from inside a strand pane is accepted only from an orch strand, which is already privileged, so a claim never raises a pane's privilege, and is refused from any other pane.
  A peer outside every pane may claim one.
  Accepted names are any upper-case name but `EXTERNAL`; the Bound says any process of the OS user outside a pane can claim one.
  The Acceptance criteria and Auto-mode assumptions gain the rule.
- F3 (fixed as proposed): D4 refuses on Linux a peer in no strand pane whose `LYX_STRAND_NAME` names a strand of this hub, and leaves the env fallback to non-Linux platforms only. The Bound now names detached processes, the Rationale claims only the in-pane bypass, and the Open risks line covers every platform for detached processes. A new criterion covers the refusal.
- F4 (fixed as proposed): D9 says daemon downtime never counts and persists the consumed TTL as an aged event, appended at liveness and pause transitions, every minute while the TTL runs and at daemon stop, so a crash loses at most one minute. D6 lists aged and has a carried inject hold its consumed TTL. A new criterion covers a restart mid-TTL.
- F5 (fixed as proposed): D12 answers a reconnecting waiter at once from the pool, handing over an undelivered reply and journaling it delivered, or reporting an inject outcome; D6 lists undelivered and delivered; the restart criterion is extended.
- F6 (fixed as proposed): D15 sweeps while unread mail or a queued inject exists and journals a gone receiver's queued injects as expired, which answers `inject --wait` and drops them from the carry; the removed-holder criterion is extended.
- F7 (fixed as proposed): D10 refuses `interrupt` and `inject --interrupt` from a non-holder while a hold covers the receiver, naming holder and expiry; the interrupt criterion is extended.
- F8 (fixed as proposed): D13 allows one hold per receiver and refuses another sender's hold, naming holder and expiry; the hold criterion is extended.
- F9 (fixed as proposed): the Acceptance criteria gain the D18 log filters and no-bodies rule, D7's 5-message cap, subject sanitizing and backoff ceiling, D4's disagreeing `LYX_STRAND_NAME`, D2's standalone refusal and D19's per-receiver independence.
- F10 (fixed as proposed): support-log.md's Q8 now reads the TTL as counted only while the receiver is live and unpaused.
- F11 (fixed as proposed): D9 reads "any sender but an orch may inject into its own strand (D16)", and the self-inject criterion adds "except an orch's".
- F12 (fixed as proposed): D9 and D16 define an orch strand as one whose role matches `orch` under `agentname.MatchesRole`, `orch-N` included; a criterion covers an `orch-N` strand.
- F13 (fixed as proposed): the plan-writer note now says no production code under `internal/` listens on a socket and points at `internal/reedengine/socketfile.go`'s dial, re-stat and remove as D3's model, which I checked against that file.
- F14 (fixed as proposed): D3's Rationale ends after "no longer holds".

## Disputed

None.

## Deferred

None.
