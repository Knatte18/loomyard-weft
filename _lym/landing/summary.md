# Verdict on Codex as assistant Code-Review reviewer, from 14 rounds

Adds `docs/research/codex/review/assist-verdict-2026-11-18/report.md`, the read of the 14 assistant reviews in the archived records of 12 landed darn runs that the trial note `codex-assist-review-trial` asked for.
No code, stencil or config changes; no review was rerun.

## What the report holds

- A per-round table: Codex's verdict, real and false findings checked against the tree, findings no Claude seat had first, Chair-kept Claude findings Codex missed, what the Chair did with each Codex finding, and fan and seat state.
- Totals: 29 real findings, 2 false (both text divergences the Chair held to be no defect), 14 unique real, 34 Claude findings missed.
- Added wait and tokens: not recorded in any archived round, and the report says so; no round shows the Chair waiting on the seat.
- Verdict proposal: keep Codex on every darn on Neve, non-gating, as a second opinion beside the Claude fan and not a replacement.
  The verdict is the operator's.

## Design decisions

- Per-round findings were read by four subagents over disjoint rounds and merged here; uncertain calls are named in the report, not folded in.
- "Unique" is counted against the Claude seats of that round only; two caveats (no lens seats in one round, lens seats that saw and declined to file in another) are stated beside the totals.
- The trial note `codex-assist-review-trial` is left as it is: the board refuses a run's change to another entry's body, so the operator closes it with the verdict.
