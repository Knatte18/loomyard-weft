# Card 7 — gitignore-pkg-delete

**Delete:**
- `internal/gitignore#`

**Prosa:**
- `docs/shared-libs/README.md`

**Intent:**
Delete `internal/gitignore` with its tests (D5): since card 6 nothing calls it, and keeping a `.gitignore` block writer around invites the next warp-side writer the new Fabric Git Invariant bullet forbids.
Remove its entry from `docs/shared-libs/README.md`'s package list.
Confirm with a grep that no Go import or doc reference to `internal/gitignore` remains.

**ImpactSummary:** Removes an unused package; no remaining caller imports it.
