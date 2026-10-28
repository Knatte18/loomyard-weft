```yaml
lens: fix-claims
outcome: reported
```

fix-claims: I checked each claim the fix added or rewrote against the code at HEAD. One finding.

- **MEDIUM**: D8's classification sentence says "a Rename or Move new side whose old side is test code". The plan model gives a Move group no new side.
  Evidence: in `internal/planparser/plan.go:215-227`, `TargetGroup` fills `Pairs` (old/new) only "when Type is CardTypeRename". `contracts/specs/loom-plan-spec.md:226` says a `Move` card's "target list holds only the symbol or file being relocated, never its new location", and the destination goes in the Intent prose.
  So the Move half of the clause names something the parser never produces. This wording came from the reviewer's F5 fix. It should cover Rename new sides only. A Move's single target is already classified as an existing glyph or file.

These claims hold:
- Quarry gives a Go struct field no glyph (`quarry/docs/glyph.md:216`).
- Quarry implements only the Go and C# alphabets.
- `shedrecipe.Env.PlanIndex` exists.
- `landingshed.Deps` and the other named Deps types are structs that `Env` nests by value.
- `walkNil` recurses only into struct-kind fields.
- The pre-resolution rejection is the `default` arm in `internal/planglyph/resolve.go`.
- `CheckEditTestFile` skips Uses, Delete, Rename and Move, and its way forward names member glyphs.
- A Rename group's `Refs` carry both endpoints.

Tool calls:
- grep of `internal/planparser` for `Pairs`/`CardTypeMove`, plus the nested Deps type declarations and `nilLegal`: my context had no Move representation and no declarations of the nested types.
- `sed` of `plan.go` and grep of the plan spec for Move: to confirm a Move group has no new side, which my context lacked.
- awk listing of `Env` field types: to check whether the nested set holds struct types outside the worktree, which my context lacked.
