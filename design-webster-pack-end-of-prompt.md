# Webster fork prompt: read instruction and kick-start pack at the end, behind a switch

`webster-pack-end-of-prompt` · Task · enhancement, webster, stencil · running · Loom-Preflight

## Seen (audit after read-all-first, 2026-10-13; board note `glyph-use-audit`)

Webster implementer forks with the kick-start pack read about 20 % less than without it, but 9 of 29 forks still read serially in their first work turn.
A Webster fork prompt runs to about 311 lines; the pack starts near line 114, and the one-turn instruction ("read every listed span in your first turn") stands only in the FRESH-READ rule, far above the pack, while the pack's own heading explains its entry format instead.

## Goal

The end-of-prompt variant of the fork prompt: the one-turn read instruction and the kick-start pack move to the prompt's last section, the instruction directly above the pack.
It sits behind a `webster.yaml` switch, like `kickstart_pack`, so it can be compared against today's layout on real runs and turned off again.

Operator decision (2026-10-15): build it behind a switch and measure it on ordinary runs rather than with a replay bench.
Orch's call, not the operator's: the switch is on in the hub default once landed, so ordinary runs measure it.

## Measuring afterwards

Not part of this task: once a few loom runs have gone through Webster with the switch on, an orch agent re-runs the audit scripts of the 2026-10-13 audit and compares the share of forks reading in parallel in their first work turn, and exploration calls per card.
To make that comparison possible, each begin-batch's existing `webster: kickstart pack` trace line also records which layout the fork prompt used.
