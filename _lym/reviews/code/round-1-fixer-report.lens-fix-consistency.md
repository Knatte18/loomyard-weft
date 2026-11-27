```yaml
lens: fix-consistency
outcome: reported
```

Task: consistency check of the fix change set against its sibling text and cross-references.

1. **MEDIUM** — F3 twin left unchanged: `pattern/PATTERN-owner-machine.md` and `PATTERN.md`
   - Changed text: `internal/boardengine/doc.go`, `# Owners`: "Board.EndBeam is the one other write of an owner".
   - The pattern background still says "Only `boardengine`'s owner setter writes either" of `owner_machine` and `prime_owner`.
   - Its Rule paragraph says "owners only on roots and beams, set only by `boardengine`'s setter". The index line for the entry in `PATTERN.md` does not carry that clause.
   - `EndBeam` clears `owner_machine` outside the setter, so the background sentence is false. The fix amended the package doc only.

2. **LOW** — F3 twin in the refusal text: `internal/boardengine/owners.go:138`
   - Changed text: the `checkOwnerPayload` doc comment, now "since an upsert never writes an owner".
   - The error that function returns still says "a board write never changes an owner".
   - `Board.EndBeam` is a board write that changes an owner, so the string now contradicts both the comment and `doc.go`.

The other siblings I checked agree with the change set: the `cli.go` package doc, the help text, the `refuseBeamCancel` and `heldBeamCancelRefusal` comments, `docs/overview.md`'s cancel paragraph, `contracts/orchrole/role-beams.md` and `_lym/landing/summary.md`.

Tool calls (2 of 6):
- A grep for other descriptions of beam cancel, `EndBeam` and the owner writer: my context held only the files the fixer had opened.
- A read of `PATTERN-owner-machine.md`, its `PATTERN.md` line, and the `role-beams.md`, `docs/overview.md` and `dobbyengine/doc.go` spans: the grep found them and none was in context.
