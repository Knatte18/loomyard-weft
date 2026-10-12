# Audit glyph use in Webster, the kick-start pack first

`glyph-use-audit` · Note · enhancement, webster

## Idea (operator, 2026-10-12)

Glyph use is probably not fully in place yet.
Analyse transcripts of a smaller task to see whether Webster uses glyphs as intended, above all the kick-start pack.

Plan's glyphs can be taken as sound for now (operator, 2026-10-12): the plan writer runs `lyx loom validate-plan` itself before it ends (step 5 of the plan stencil), the twill lens `forced-edits` checks each listed glyph against `lyx quarry`, and with twill a failed self-check cost one turn in three of six runs.

## What to check in Webster

- The kick-start pack (`internal/websterengine/doc.go`, "the kick-start pack"): how many refs resolved and how many did not (the `webster: kickstart pack` Info line per begin-batch), and why the unresolved ones failed.
- Whether the implementer fork reads the listed spans in its first turn as FRESH-READ says, or re-reads whole files, greps or runs `lyx quarry` for members the pack already resolved.
- Whether a fork edits outside the listed spans, and whether that work rests on stale reads.
- The `deviations` a fork reports against what `ScopeGuard` finds (`webster-deviations-list`).

## The fork prompt (operator, 2026-10-12)

A Webster fork prompt (`reed-multi-window`, batch 01) runs to about 311 lines; the kick-start pack starts near line 114.
The one-turn instruction stands only in the FRESH-READ rule, far above the pack; the pack's own heading explains its entry format instead, and step 1 restates FRESH-READ.
The operator doubts the pack has any effect as it is delivered today.

Idea to test once the audit shows how forks read: two prompts instead of one.
The first carries the background (role, rules, cards, gates, report); the second says only "read all of this in one turn, in parallel, before any edit" followed by the pack.
That costs a turn, but the read is then the only thing the fork is told to do in that turn.
Open: how a fork spawned through the Agent tool receives a second prompt.

## Seen

`webster-deviations-list`: a Webster fork's `deviations` list mixed in files that probably house its batch's own target glyphs, because the implementer stencil asks it to compare against the glyph union and also tells it not to.
