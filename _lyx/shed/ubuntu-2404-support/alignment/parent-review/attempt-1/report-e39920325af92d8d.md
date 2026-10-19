---
verdict: APPROVED
---

## Scope check against the board entry

Every part of `ubuntu-2404-support` has a decision, and none reaches past the entry.

- #739, socket path under a 10-digit uid: D1, the six-hex run directory name from the #745 diff, landing first as the entry requires.
- #743, `cherry-pick --empty=drop`: D2 is the operator's decision word for word, a plain pick and `--skip` on an empty pick with HEAD unmoved; the two Bolt tests and the merge-lock failure the entry lists under #743 are covered by D2 and D7.
- #738, #740, #748, reed's session row: D3 draws each session once with no comma and the prime leftmost, and its acceptance checks the row rendered, as #748 asks.
- #744: D6 diagnoses and fixes the cause, per the operator's decision relayed in the support log.
- #747 audit: D5 audits every git and tmux invocation and records the result.
- #747 minimum-version check: D4 refuses a below-floor git or tmux with an error naming the tool, the version found and the floor.
  The entry's proposal says "at start-up"; D4 checks once per process at each tool's first use instead, and states that it replaces the wording.
  The entry's point, a host below the floor failing plainly instead of in odd places, holds, and no operator decision fixes the timing, so this is a design choice for the review segment, not a scope deviation.
- Proof: the acceptance criteria run both gate tiers here, and the run on the NORCE host is left to the operator after landing, as the operator decided.

The Out list matches the entry and the board: `run-machine-ownership` and `open-window-per-machine` stay out per the operator, and `xdr-git-test-file-burst`, `pause-background-work` and `doctor` are kept out.
D4's floor check is no `lyx doctor` verb, so it does not duplicate `doctor`, which describes a layout, config, board and lock diagnostic.
D4 logs reed's probe spawn under the existing spawn rule and adds no audit log, so it does not take work from `spawn-audit-log`.
D3 leaves the follow button and the time in place and adds no action buttons, so it does not touch `reed-status-buttons`.
D7 names XDR-slowed git spawns as the likely cause of the merge-lock flake and fixes the test's synchronization only; reducing the fixtures' file burst stays with `xdr-git-test-file-burst`.

## Overlap with peer runs and parent commits

`lyx loom peers` lists no landed parent commits and one peer run.

- `hub-workspaces` (row 7, own row 3): both runs edit `internal/fabricengine`, at different places.
  This run changes `dropSeedCommits` and its `doc.go` paragraph and adds a lock-wait test seam; `hub-workspaces` adds the hub workspace regeneration, retires `_portals` and changes two other PATTERN entries.
  No behaviour, decision or PATTERN entry is shared, so this is a text-only overlap.
  Landing order: `hub-workspaces` is further along (row 7 against row 3), so it lands first and this run takes its changes at Merge-In.

Bookkeeping only: `docs/overview.md`'s module tree gains `internal/toolfloor`, and no PATTERN index line is added.
