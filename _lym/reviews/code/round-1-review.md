---
verdict: APPROVED
---

Range reviewed: `git diff bd707e161e7762c93fa52735b7280fa29bccf227..HEAD`, one commit (`5d66d695`) touching `internal/heddleengine/shipped.json` and four test files.
Measured against the board entry under `## Goal` in `_lym/discussion/decision-record.md`, which has no `Added after Discussion` entries.
`lym gate test ./internal/heddleengine` tier 0 passed on this tree (139 passed, 0 failed).
All three lenses reported no findings, and the holistic review found none.

## Against the board entry

- **Aim both entries at `agents-file`:** done in `shipped.json`. `reachesFileTarget` and `targetTable`'s `agents-file` row carry both entries into `CLAUDE.local.md` and `AGENTS.override.md`.
- **Reword the lines that name Claude Code's tools:** all three phrases the entry names are reworded.
  "even though the Read tool advises" became "even when your read tool advises", "every Read and grep call" became "every read and search", and "every Edit and Write call" became "every file edit".
  Each rule keeps its meaning.
  The remaining lowercase "grep" in "Send every grep and read" and "Never grep for a line number" names the shell command, which a Codex session also runs, not a Claude Code tool, and the entry does not list those lines.
- **Update `ShippedEntriesArePinned` and the tests that pin the rendered files:** done. `BothFilesHoldTheShippedLayerWithNoEntry` checks that a rendered `AGENTS.override.md` holds both entries, both by exact equality and by heading.
- **Root `CLAUDE.local.md`:** untracked, and it follows on the next render. The diff rightly leaves it alone.
- No change goes beyond `shipped.json` and the tests that pin its render.
  `PATTERN-heddle-instructions` holds: lym's instructions stay in the shipped layer and render only through `heddleengine`.
- `_lym/landing/summary.md` matches the diff, and the commit message carries no `Pattern-Departure:` trailer.

## Focus questions

- **Wording in `shipped.json`:** covered above; not a defect.
- **The non-parallel `TestRenderRemovesAStaleMarkedRenderEvenOffExcluded`:** `useShippedLayer` restores `shippedStore` through `t.Cleanup`.
  Go resumes paused parallel top-level tests only after every sequential top-level test has finished and run its cleanups, and `TestRender` reads `shippedStore` only after `t.Parallel()`.
  This is the same pattern `TestShowAndPassagesReadTheShippedLayerFirst` and the settings tests already use. Not a defect.
- **`fileTargetWant`:** it decides which entries to expect from each entry's own `Targets` and the literal `agents-file`, not by calling `reachesFileTarget`.
  So a production bug that dropped shared entries from either file would still fail the comparison. Not a defect.
- **The prefix and suffix check in `TestSeam_PullTakeIn`:** it pins the opening `AGENTS.md` text and the trailing store entry, so the order holds.
  The external test package cannot reach `shippedStore`, and the unit tests in `render_test.go` pin the exact shipped section. Not a defect.
- **Test economy:** the moved stale-render test is the only remaining coverage of removing a marked file whose target has no entry, since the real shipped layer now gives every file target text.
  The redundancy report lists it as unique. The heading loop is covered under Rejected below.
- **`overview_test.go`:** both shipped text entries now name only `agents-file`, so "- From: shipped with lym" belongs in that section. The tier-0 run confirms it.

## Rejected

- The heading assertions in `BothFilesHoldTheShippedLayerWithNoEntry` duplicate coverage: rejected.
  `ShippedEntriesArePinned` together with the exact `codexFileWant("")` comparison pins the same thing, but the board entry asks for this check.
  The two headings are the entries' identity rather than prose the rule is about.
  A finding to delete the check would contradict the answer key.
- `TestSeam_PullTakeIn` is weaker than the earlier exact match: rejected.
  The test's subject is the take-in render's trigger and opening file, the shipped section's exact text is pinned at tier 0, and an exact match would need the unexported shipped layer.
- `fileTargetWant` adds `shippedHeading` even when no shipped entry reaches the target, while production omits it: rejected.
  No caller reaches that case under the real shipped layer, and the helper's behaviour there predates the diff.
