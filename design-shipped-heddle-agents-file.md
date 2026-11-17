# Give Codex lym's shipped heddle instructions too

`shipped-heddle-agents-file` · Task · bug, heddle, codex · running · Code-Burler-Fix

## Seen (operator, 2026-11-17)

lym's two shipped text entries in `internal/heddleengine/shipped.json`, `lym-batching` and `lym-plan-edits-before-gate`,
target `claude-file` alone, so they render into `CLAUDE.local.md` and never into `AGENTS.override.md`.
A Codex session gets neither, while it should get the same instructions as a Claude session.

## Fix

- Aim both entries at `agents-file`, so they render into both files.
- Their text names Claude Code's tools: "the Read tool advises", "every Read and grep call", "every Edit and Write call".
  Word those lines so they hold for both agents,
  such as "every read and search", "every file edit", "even when your read tool advises reading only part",
  keeping each rule's meaning.
- Update `ShippedEntriesArePinned` in `internal/heddleengine/store_test.go` and any test that pins the rendered files,
  and check that a rendered `AGENTS.override.md` holds both entries.
- The root `CLAUDE.local.md` of this repository carries the same two sections, rendered from the store; it follows on the next render.
