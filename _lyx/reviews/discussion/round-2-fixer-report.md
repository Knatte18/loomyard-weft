# Fixer report — discussion round 2

## Changes

- **F1 (BLOCKING), fixed as proposed, option (a).**
  Premise confirmed: `evaluateGate` in `internal/shuttleengine/wait.go` reports every entry after a failure `GateEntryNotReached` without calling it, so `parentreview`'s `final` (`internal/parentreview/gate.go:271`) never runs behind a failing `discussion` entry.
  D2's Decision now says finalize reaches `Final` only when every earlier entry passes.
  D2's Bound drops "fires only where the at-boundary gate would itself pass" and "re-evaluates every entry".
  It now states that the probe reports expiry whatever the tree's state, and that a failing tree ends `Stuck` with the request open, unexpired and with no timeout Warn.
  It also states what follows: a resume marks the request expired at its first boundary whose tree passes, through `gate()`'s existing expiry branch (`gate.go:173`), and a fresh spawn's `PrepareRound` begins a new round, leaving the old request open and unread, since the gate reads only `Latest()`.
  The matching acceptance line now says `Stuck` with the request open and no timeout Warn.
  Option (b) was not taken: it would change `evaluateGate`'s stop-at-first-failure rule for every gate, and the support log's Rejected alternatives already accept the earlier halt.
- **F2 (MEDIUM), fixed as proposed.**
  Premise confirmed: `parentreview.Delivery` (`store.go:69`) has no stamper field, and `internal/loomcli/review.go:9` says the verbs check no caller.
  D11's Decision asks for only a delivered-at and a later verdict, and its landing caveat now also says neither file records who stamped the delivery or that a SendMessage preceded it.
  The Rationale names the `delivered` verb's missing caller check as a second gap.
  The acceptance line and the D11 Open risk say the same.
- **F3 (LOW), fixed as proposed.**
  Dropped the scope-only Auto-mode assumption and the first Open risk, and resolved the ledger line as decided by the operator after Discussion.
- **F4 (LOW), fixed as proposed.**
  The ledger answer now says the interval is the writer's cache lifetime less one minute, told through `GateConfig`, and only the cap of 3 is a constant in `internal/parentreview`.
- **F5 (NIT), fixed as proposed.**
  Dropped the `lyx shuttle await` clause from D1, since `DefaultAwaitCap` is 250 seconds (`internal/shuttleengine/await.go:15`), and changed "never pinged" to "not pinged at the default 60-minute bound" in D1 and its acceptance line.

## Disputed

None.

## Deferred

None.
