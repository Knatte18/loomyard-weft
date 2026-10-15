---
verdict: APPROVED
---

Review range: `git diff $(git merge-base main HEAD)..HEAD` (no rework round exists; `product.parent` is `main`), four commits covering cards 1 to 4.

Each focus question settles as follows.

- **Card 1.** `Config.KickstartPackPerCard` carries the tag `kickstart_pack_per_card`, `ConfigSpec` registers it as `CategoryNone`, and `template.yaml` sets it `true`, so a layer file lacking the key loads as true.
  `TestLoadConfig` pins the override round-trip and a non-boolean value, and `TestDispatchLayers` pins `--set kickstart_pack_per_card=false`.
- **Card 2.** `CardPackMarkdown` only reads (stat of the plan directory, `ParsePlan`, `index.KickstartPack` against the worktree root), so the spans are the ones at call time, uncommitted edits included.
  A non-integer or `0` argument, an unknown card, a missing plan directory and an unparsable plan each fail with `CardPackWayForward`, and the four rows are in `contracts/specs/refusal-spec.md`.
  `TestCardPackCmd` covers each refusal and the uncommitted-edit span shift.
- **Card 3.**
  - `packLayoutOf` is the single choice point.
    A nil pack ignores the layout, and the recovery prompt passes `PackLayoutFile` with `packOn` false, so the pack-off and recovery bytes stay.
    The file and inline fragments change only in their banners.
  - `buildKickstartPack` logs `pack_layout=card` only when a pack is built and the switch is on in a hub geometry.
  - Each rendered `card-pack` call logs `webster: card pack`.
  - The step 1 fragment places `lyx loom card-pack <N>` before the card's first edit, and the `fresh_read` fragment puts the card-file read in the first work turn.
- **Card 4.** `tools/forkaudit` gains the `card` arm and the `card packs` column.
  The existing arms' spans and nets are computed as before, and the integration test passes.
- **Test economy.** The new tests each pin a behavior no existing test covers.
  `cmd/lyx` passes, so the test budget holds.

Verification run: `lyx gate test` over `internal/websterengine`, `internal/loomcli`, `internal/configcli`, `contracts/stencils` and `cmd/lyx`, and `lyx gate test --tags integration ./tools/forkaudit`, all green.
No card carries a `Rename` or `Delete` group, so no per-type mechanical check applies.
No finding.
