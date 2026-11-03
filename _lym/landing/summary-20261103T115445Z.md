# Heddle: shipped instructions as headed imperative bullets

The two shipped heddle entries, `lym-batching` and `lym-plan-edits-before-gate` in `internal/heddleengine/shipped.json`, used to render into every worktree's `CLAUDE.local.md` as solid paragraphs of prose.
They are now short imperative bullet lists, one instruction per bullet, so agents read them as rules.

## What changes

- Each entry opens with a `##` heading of its own: `## Batch your calls` and `## Make every round of edits in this order`.
  The headings sit under `# LoomYard's own instructions`, so the rendered file skips no heading level, the same way a repository entry's `##` heading sits under `# This repository's instructions`.
- `lym-batching` gives each of its instructions as its own bullet: batch the calls you know, list the questions first, wait only to choose the next call, read a file whole or in all its spans in one message over the Read tool's advice, never grep for a line and then read a slice, keep read files in mind, and read again every file a command rewrote.
- `lym-plan-edits-before-gate` keeps the round's four steps as a numbered list.
  Step 1's second-round rule and step 4's fix cycle become sub-bullets.
  The closing prohibitions stay as bullets.
- "Never read one file in slices over several turns" now appears only in `lym-batching`, where it already stood.
  The round entry no longer repeats it.

## Design decisions

- The heading goes in each entry's text, not in the renderer.
  A repository entry carries its own `##` heading in the same way, and a lym override replaces the whole text, heading included.
  The renderer in `render.go` is unchanged.
- The `ShippedEntriesArePinned` subtest in `internal/heddleengine/store_test.go` no longer pins the whole `lym-batching` text, which only duplicated `shipped.json`.
  It now pins each entry's opening heading line, which is the structural contract that keeps the rendered file free of heading-level skips.
  The render tests build their expected file from the shipped layer itself, so they follow the new text unchanged.
- `docs/research/turn-audit-2026-10-28/report.md` quotes an older form of the batching sentence.
  It is a measurement report of a past run and is left as written.

## Docs

The `heddleengine` package doc now says that each shipped entry opens with a level-two heading under the shipped section's level-one heading.
