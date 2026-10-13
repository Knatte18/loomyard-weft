---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: _lyx/landing/summary.md:21
    summary: The diff puts the pack in a separate file where the board entry asks for it inline as the prompt's last section, and the summary's "operator's choice" claim rests on the HEAD commit message alone
    fix: In _lyx/landing/summary.md, replace the sentence "the operator's choice, so the prompt's last line is a single instruction..." under "## Design decision" with one that states the departure from the board entry plainly (the entry asks for the instruction and pack inline as the prompt's last section) and names commit d8d1c0956 ("restore the kick-start pack's own file, per the operator's decision") as where the decision is recorded. No code change.
  - id: F2
    severity: NIT
    class: consistency
    location: contracts/stencils/webster/webster-fragment-pack-card.md:1
    summary: Banners of the pack card, unresolved and empty fragments still say they belong to the fork prompt, but they now render into the pack file
    fix: In webster-fragment-pack-card.md line 1 change "kick-start pack in webster's fork prompt" to "kick-start pack file of a webster fork"; in webster-fragment-pack-unresolved.md line 1 and webster-fragment-pack-empty.md line 1 change "in webster's fork prompt" to "in webster's kick-start pack file". Keep the banner-stripping behavior untouched.
---

### [MEDIUM] Pack in its own file departs from the board entry, and the summary's authority claim is thin

**Location:** `_lyx/landing/summary.md:21` (the "Design decision" paragraph), against `contracts/stencils/webster/webster-fragment-pack-read.md` and `internal/websterengine/render.go` (`RenderForkPrompt`)

**Issue:** The board entry says "the prompt's last section is the one-turn instruction ... followed by the pack".
The diff does not do that: the prompt ends with a one-line `## Start here` section that names `<batch>-pack.md`, and the instruction and pack live in that file.
The entry was not changed, and nothing in the repository records the decision except the HEAD commit message ("per the operator's decision", which reverted two earlier commits that had made the pack inline).
The summary presents the departure as "the operator's choice" without saying it differs from the entry or where the decision is recorded.
The code itself is sound for the layout it implements, so this is a scope and traceability finding and not a defect in the change.
The settled points from the focus directive:

- The no-pack prompt is byte-identical to before: the `kickstart_pack` marker moved to the body's final line, which keeps its trailing newline, and the nil-pack render test compares against a rebuilt pre-pack body.
- Without a pack no file is written, and the recovery prompt never takes a pack.

### [NIT] Fragment banners still place the pack subsections in the fork prompt

**Location:** `contracts/stencils/webster/webster-fragment-pack-card.md:1`, `webster-fragment-pack-unresolved.md:1`, `webster-fragment-pack-empty.md:1`

**Issue:** Their banners read "in webster's fork prompt", but these fragments now fill the pack file only.
The pack section and fresh-read banners were updated; these three were missed.

## Settled without a finding

- **Empty pack.** The entry lists "empty pack" among the cases where the prompt reads as today, so `kickstart_pack=empty` is licensed.
  `buildKickstartPack` returns nil for it, so the prompt, the pack file decision and the trace line all follow the same nil.
  The recovery prompt never takes a pack, and the begin-batch envelope carries none, so no other path renders one.
  The new "no entry on any card" row extends `TestBeginBatch_KickstartPack`; no existing test covers that state.
- **Stencil layout.** `{{.kickstart_pack}}` ends `webster-body-implementer.md` after the batch-report section, which still names the report as the fork's last action; `claims_test.go` pins the order.
  The pack-on FRESH-READ and locate fragments carry no read instruction and point at the pack file.
- **Pack file lifecycle.** `packPath` is absolute and beside the prompt file, so the fork can open it.
  The pack file is written before the prompt, and a write failure returns an error as the prompt write does.
  `Run --fresh` removes the whole prompts directory, so cleanup covers both files.
- **Docs and budget.** `internal/websterengine/doc.go` describes the pack file, the read line and the `empty` state.
  The diff adds no top-level test, so `cmd/lyx/testdata/test-budget.yaml` is unaffected.
  The render tests assert the layout (prompt ends with the read line, file opens with the instruction) and overlap neither `TestBeginBatch_KickstartPack`'s file and trace assertions nor each other.
