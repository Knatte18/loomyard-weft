```yaml
lens: fix-claims
outcome: reported
```

Task: check that every claim the fix added or rewrote holds against the code at HEAD (fix-claims lens).

**MEDIUM: D9's new sentence says commit c8e43ac4 changed both files, but it touched only one.**
- **Changed text:** D9, "The rule this rewrites is the Motif-first rule that commit c8e43ac4 landed on main, so the run takes main's text of both files before editing them."
- **Evidence:** `git show --stat c8e43ac4` lists only `plugins/ly/skills/board-policy/SKILL.md` (8 lines changed, 7 insertions, 1 deletion). The commit did not change `contracts/orchrole/orch-role.md`, so main's text of that file does not hold the Motif-first rule.
- **Origin:** this claim comes from the reviewer's F6 fix.
- **Correction:** name only `plugins/ly/skills/board-policy/SKILL.md` as the file to take from main.

The other claims I checked hold:
- **Subject seam:** a Bouncer with no Subject seam clears on entry over a settled verdict (`settledEntry`, bouncer.go ~782). The clear archives the run directory aside.
- **Soft-cap rule:** with AutoAccept off, an open window reaches `softCapRule`. That rule auto-accepts a round with no design-wrong keys, records the automatic accept, and otherwise continues or escalates at the hard cap or on circling. A round with no `carry_over` carries nothing into Open risks.
- **`freshRaddleEntry`:** it counts both a return and a goto routed to Raddle-Write as a fresh entry.
- **Producer name:** `loomshed.NameRaddleFile` is "Raddle-File", and `RunStatusProducer` exists for the new admission check.
- **Clearing the status:** a converted, non-beam project with a run status is held by `RunHolds`, and clearing only its Status passes `scopeChanged`.
- **Mark timing:** a failed store write returns an error from `FileRaddle` after the save, so a mark written after `FileRaddle` returns is not yet on disk. The new D6 text states that window correctly.
- **c8e43ac4:** it is on main and not yet in HEAD.

Tool calls:
- One Bash call read `settleUndecided`, `git show --stat c8e43ac4` and an ancestor check against main, the value of `NameRaddleFile`, and `RunHolds`. My context did not have the soft-cap window entry, the commit's file list, the producer constant or the hold rule.
