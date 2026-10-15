---
round: 1
focus:
  - "D2/D3: the inline section is placed by a marker at the end of the per-card closing paragraph, before `## The sandbox rule`; settle whether that marker position and the leading blank-line separation reproduce the `1e39afc10^` placement, and whether D3's 'other fills empty and leaves no byte' holds for the body's separators in both layouts."
  - "D3/D4: the pack-off and recovery prompts must render today's bytes; settle whether the decisions state every template that shares `webster-body-implementer` and must fill or list the new marker, so no render path leaves it unfilled."
  - "D1/D4: `kickstart_pack_file` is a new switch; settle whether the Decisions state what it can skip or let through (no pack file written, no `## Start here`, a stale `<batch>-pack.md` left) and what bounds it, per the attack-surface question."
  - "D5: settle whether `pack_layout` values (`file`, `inline`, `none`) cover every path that emits the `webster: kickstart pack` line, including the degraded-pack and empty-pack paths, and whether the placement after `unresolved` is consistent with the stated log pins."
  - "Acceptance criteria: settle whether each criterion is verifiable from the Decisions as written (byte-identity under the file layout, text equality to `1e39afc10^` with banners stripped, flip on re-run) and whether any Decision lacks a criterion that would catch its failure."
---

Premises worth checking first: that `1e39afc10^` is the layout the audit measured, and that the empty-pack departure in D4 is bounded as the Auto-mode assumptions say.
