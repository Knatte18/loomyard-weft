# Render lym's shipped heddle text into AGENTS.override.md too

A Codex session never received lym's two shipped text entries, `lym-batching` and `lym-plan-edits-before-gate`.
Both targeted `claude-file` alone, so they rendered into `CLAUDE.local.md` and never into `AGENTS.override.md`.
Both now target `agents-file`, so every render writes them into both files, and a Codex session gets the same instructions as a Claude session.

## Changes

- `internal/heddleengine/shipped.json`: both entries move from `claude-file` to `agents-file`.
- The entries' text named Claude Code's tools, so it is reworded to hold for both agents with each rule's meaning kept:
  - "even though the Read tool advises" becomes "even when your read tool advises";
  - "every Read and grep call" becomes "every read and search";
  - "every Edit and Write call" becomes "every file edit".
- The repository's root `CLAUDE.local.md` carries the same two sections, rendered from the store, and follows on the next render.

## Tests

- `ShippedEntriesArePinned` in `store_test.go` pins the new target.
- `render_test.go` gains `codexFileWant`, which builds the expected `AGENTS.override.md` from the shipped entries aimed at `codex-file` or `agents-file`.
  The render cases now expect the shipped section in `AGENTS.override.md` and list it in `report.Written`.
  A case checks that a rendered `AGENTS.override.md` holds both shipped headings.
- The stale-render removal case no longer holds with the real shipped layer, because every file target now has shipped text.
  It becomes `TestRenderRemovesAStaleMarkedRenderEvenOffExcluded`, which swaps in an empty shipped layer.
  It is not parallel, because it replaces `shippedStore`, which every render reads.
- `overview_test.go` expects the "shipped with lym" source line under `agents-file` rather than `claude-file`.
- `seam_integration_test.go` checks the pulled `AGENTS.override.md` by prefix and suffix, since the shipped section now sits between the anchor's `AGENTS.md` text and the store's entry.

## To check by hand

- No test run is recorded for this change, so run the `internal/heddleengine` tests, including the seam integration test.
- Render a worktree and confirm its `AGENTS.override.md` holds both shipped sections.
