---
format: 5
approved: true
language: go
---

# Plan: creel mail delivered once

One creel mail reaches the model once and the operator once, as one message, with no empty turn, no error label and no hook command text, and each hook delivery attempt is logged.
The Stop and tool-boundary outputs stop being blocks (D1, D2), the idle ring hands the mail over itself with creel's wake labels (D3, D4), the operator gets the ring's mail as an echo at the woken turn's first claim (D6), and the daemon logs every claim and bell (D5).

## Card Index

1 — hook-output-seam — replace the boundary block with additional context and the system message (D1, D2)
2 — waiter-rewake-labels — set rewakeMessage and rewakeSummary on the waiter hook (D4)
3 — ack-failure-releases — a failed journal write at a claim's acknowledgement releases the ids it left unjournaled (D3)
4 — ring-claims-mail — the daemon claims mail into the ring of a marked waiter and awaits its acknowledgement (D3)
5 — client-ring-ack — the Go client never resumes a hook-wait whose ring reached its handler and names an unsent ack (D3)
6 — hook-wait-hands-mail — hook-wait writes the ring's lines as the wake and acknowledges them (D3)
7 — ring-echo — an acknowledged ring leaves an echo the next claim hands out ahead of the mail (D6)
8 — hook-read-renders-echo — hook-read shows the echo as the system message, and an echo-only Stop exits 1 (D6)
9 — claim-delivery-log — one daemon log line per claim outcome and per bell send (D5)
10 — smoke-hook-delivery — the tier3 real-session checks of D1, D2, D3 and D6

## Shared Decisions

### Decision: seam-method-replaced

- **Decision:** `ToolBoundaryContextRenderer.BoundaryBlock` is replaced by `HookOutput(event, mail, echo)`, created beside it and the old one removed in the same card, since the hub has no `rename_tools` and the output is no longer a block.
- **Rationale:** the decision record's note expects the rename; the echo parameter lets one engine method render D1, D2 and D6, keeping every Claude key in claudeengine.
- **Applies to:** cards 1-hook-output-seam and 8-hook-read-renders-echo

### Decision: release-on-cancel

- **Decision:** a ring outcome left untaken in the waiter's channel is released by `Waiter.Cancel`, which every end of `awaitOutcome` runs, rather than by a test seam that orders the outcome before the close, and a ring response whose write fails goes on to `daemon.awaitAck`, whose failed read of the ack line releases it, so the ring path has two release steps, each reached by a row.
- **Rationale:** the release then holds on every end of the answer path, and a store-level row reaches it deterministically by cancelling a rung waiter without reading its channel.
- **Applies to:** card 4-ring-claims-mail
