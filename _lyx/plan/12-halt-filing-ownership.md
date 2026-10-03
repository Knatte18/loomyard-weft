# Card 12 — halt-filing-ownership

**Prosa:**
- `contracts/stencils/friction/friction-template-reflection.md`
- `CONSTRAINTS.md`
- `docs/overview.md`

**Edit:**
- `contracts/stencils/claims_test.go`

**Intent:**
Tell the reflection agent and the docs which path owns which filing, so one halt is filed once, and record that Tier 1 filing runs under `step` and retries a failed filing (PR-Gate F1 and F2).

Reflection stencil, keeping the no-conditionals rule and its four markers:

- Step 2 gains the sentence `When a halt note says Tier 1 anomaly filing files the halt event itself, do not file the halt event; file only the lyx problems behind it.`
  It quotes the line card 11's `writeHaltNote` adds when the `selfreport` knob is on, `Tier 1 anomaly filing files the halt event itself; file only the lyx problems behind it.`; a halt note without that line is grouped like any other note.

`claims_test.go`'s `friction-template-reflection.md` row gains a claim on that sentence's `do not file the halt event; file only the lyx problems behind it`, its `why` naming Tier 1 as the halt event's filer.

`CONSTRAINTS.md`'s Friction Capture Invariant gains one clause: with the `selfreport` knob on, Tier 1 anomaly filing files a halt event under `run` and `step` alike, retrying a failed filing on every later pass, and the halt note says so; the reflection files only the lyx problems behind the halt.
Its enforcement line adds the `selfreport_test.go` and `halt_test.go` cases cards 10 and 11 add.

`docs/overview.md`'s selfreport bullet replaces "files them after every `lyx loom run` call" with filing after every `run` and `step`, and adds that an anomaly whose filing failed stays pending in the filed-title marker and is retried on every later pass.

What this lets through, and the bound: the reflection now leaves the halt event itself to Tier 1 whenever the halt note says so; it still files every lyx problem behind the halt, and a halt note written with the knob off carries no such line, so with Tier 1 off the reflection files the halt as before.
With the knob on, a halt event whose Tier 1 filing failed is left to card 10's pending retry on the next Tier 1 pass, one whose pass was skipped is filed by the next pass only while the run is still halted, and neither is filed if no later `run` or `step` runs in that worktree; card 11 states the same residual.

**ImpactSummary:** the reflection stencil's text, one claim row entry, and prose in `CONSTRAINTS.md` and `docs/overview.md`; no Go behaviour changes.
