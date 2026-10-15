---
round: 2
focus:
  - "Card 6 moves `internal/loomcli/drive.go` into `internal/sheddriver`, and `resumeOpenAttempt` reads an open arbiter attempt via `loomengine.ReadArbiterAttempt`: settle whether the moved code still imports `loomengine` or whether some seam carries that read, and whether a test pins the restart of a halted run whose only open attempt is an arbiter attempt."
  - "`tacklerCall.settleFresh` in the tackler call that card 4 moves calls `TacklerDeps.Snapshot` unconditionally, while card 6 declares `Snapshot` optional and card 13 wires none: settle which card owns the nil-safety and whether a test reaches a spawn with a nil `Snapshot`."
  - "Card 13 adds `battenDriveRefusal` reachable through `lyx batten run`: settle whether a `contracts/specs/refusal-spec.md` row lands with it, naming a way forward and an input and test that reach it through the verb."
  - "The acceptance criteria require `driver_kind` in `batten.yaml` to be refused: settle whether the refusal covers the hub base layer as well as the local layer, given `configengine.ValidateLocal` runs on the local layer only."
  - "Check the plan's new fixes against the attack-surface rule and card order: every new seam or refusal states what it can skip and what bounds it, and no card depends on a later one."
---

Focus entries restate the sites they depend on and make no claim about severity or outcome.
