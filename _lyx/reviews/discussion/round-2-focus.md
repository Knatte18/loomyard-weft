---
round: 2
focus:
  - "In decision-record.md, check that D3's writes-nothing claim, D4's Bound and the failing-regeneration criteria agree with D5's in-place write, which truncates before it writes (`os.Root.WriteFile` in `internal/fabricengine/hubworkspace_write.go`): settle what a failed or killed write leaves behind and what repairs it."
  - "In decision-record.md D5, check that the lock is fully placed and timed: its path under `<hub>/_board/.lyx`, its wait value, and what a regeneration does on a hub with no `_board` (no failure, no created directory, no `Bolt` listed), against `lock.AcquireWriteLockWithin` creating no directories."
  - "In decision-record.md D2, D4, D6 and D7, check that every new bound introduced by the fixes is stated: a directory at the retired workspace path `<hub>/_launchers/<AnchorRel>/<prime>.code-workspace`, a prime branch change via `Topology.Checkout` against the display name D2 derives from the checked-out branch, and any hub-level detail field the Decisions rely on against the result types in `internal/fabricengine`."
---

Round 2 should judge the revised text fresh, including any text the fixes added.
