# Webster fork prompt: switch between the pack-file layout and the inline layout

`webster-pack-layout-switch` · Task · enhancement, webster, stencil · running · Plan-Write

## Seen

The audit of 2026-10-13 (board note `glyph-use-audit`) measured Webster implementer forks on the layout of `113593990`: the kick-start pack inline in the fork prompt, starting near line 114 of about 311, with the one-turn read instruction only in the FRESH-READ rule at the top.
9 of 29 forks still read serially in their first work turn.

Since `1e39afc10` (2026-10-13), following the operator's decision in `d8d1c0956`, the fork prompt ends with a one-line `## Start here` section that sends the fork to `<batch>-pack.md`, which holds the one-turn read instruction and the pack.
That layout is always on whenever a batch has a pack; no switch selects the earlier layout, so the change cannot be compared on ordinary runs.

## Goal

A `webster.yaml` switch that picks the fork prompt's pack layout, beside the existing `kickstart_pack` switch, which keeps turning the pack on and off:

- On (hub default): today's layout from `1e39afc10`, pack file plus the closing read line.
- Off: the layout the audit measured, as it rendered at `113593990`: pack inline in the prompt, read instruction in FRESH-READ, no pack file.

Each begin-batch's existing `webster: kickstart pack` trace line also records which layout the fork prompt used, so a later audit can split forks by layout.

Operator decision (2026-10-15): build the switch; the task is small and runs on a low review budget.

## Not part of this task

Measuring: once runs have gone through Webster with each layout, an orch agent re-runs the 2026-10-13 audit scripts and compares the share of forks reading in parallel in their first work turn, and exploration calls per card.
