# Rewrite C13 for window-per-strand and add the creel re-test report

Docs only; no code changes.

## What changed

C13 in `tools/sandbox/SANDBOX-CREEL-SUITE.md` described its second half against the removed stacked layout and `collapsed_rows`, a setup that can no longer be built.
The half now adds a `lyx reed add --cmd bash` strand, which reed places in a window of its own and whose pane shows no input box.
The scenario checks that creel reports the `no-input-box` stall as a `stalled` event in `lyx creel log` and as a progress line on stderr of `lyx creel inject --wait`.
The heading changes from "Collapsed and hand-added strands" to "Hand-added and stalled strands".
The hand-added Claude half is unchanged.

`docs/research/creel-audit-2026-10-17/report.md` is the creel stage 1 re-test report, copied byte for byte from the operator's scratch file.
It measures one run on `58a7feec7` and stays as written.

## Design decisions

- The bash strand provokes `no-input-box` because that stall is reproducible by the agent; `too-short` now arises only from a short viewer terminal, and `draft` needs a Claude strand plus operator help.
  The rewritten half therefore does not exercise `too-short` or `draft`.
