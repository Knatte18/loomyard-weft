# Give Codex lym's shipped heddle instructions too

lym's two shipped text entries, `lym-batching` and `lym-plan-edits-before-gate`, targeted `claude-file` alone.
They rendered into `CLAUDE.local.md` and never into `AGENTS.override.md`, so a Codex session got neither.
Both now target `agents-file`, the shared target that renders into both files,
so every worktree's `AGENTS.override.md` opens its heddle section with the same `# LoomYard's own instructions` block a Claude session reads.

## Wording

Three lines named Claude Code's tools and now hold for both agents with the same meaning:

- "even though the Read tool advises reading only the part you need" became "even when your read tool advises reading only the part you need";
- "every Read and grep call together" became "every read and search together";
- "every Edit and Write call together in one turn" became "every file edit together in one turn".

## Tests

- `ShippedEntriesArePinned` pins both entries at `agents-file`.
- The render tests build the expected `AGENTS.override.md` from the shipped layer the same way they already built `CLAUDE.local.md`.
  A render of an empty store now writes both files, and `BothFilesHoldTheShippedLayerWithNoEntry` checks that `AGENTS.override.md` holds both shipped sections.
- With the real shipped layer every file target now always has text, so the stale-render removal case moved to its own non-parallel test over an empty shipped layer.
- The overview test finds the shipped entries under the `agents-file` section, and the seam integration test checks the take-in render's opening and closing text around the shipped section.

## Follow-on

This repository's root `CLAUDE.local.md` is rendered from the store and picks up the new wording at its next render.
