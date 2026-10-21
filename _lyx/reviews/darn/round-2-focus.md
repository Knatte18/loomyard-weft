---
round: 2
focus:
  - "internal/boardengine/doc.go `# Intake`, internal/boardengine/intake.go header and internal/boardengine/task.go `Issues` field comment: does any wording still say GitHub is the selfreport inbox, or lists inbox issues, now that intake reads the hub's own GitHub repository?"
  - "internal/boardcli/cli.go `hubRepo` closure: do the boardcli tests exercise its nil-location refusal, and show that a non-intake verb run through `Command()` never resolves the repository?"
  - "internal/boardcli/intake_test.go: do the `import` and `close` handlers each have a case for an unresolvable hub repository, asserting an error envelope and an unchanged board.json?"
  - "internal/fabricengine/hubrepourl_integration_test.go: does the neither-source step assert that the error names both the warp binding file and the origin remote?"
  - "internal/orchcli/doc.go step 5: does the friction-findings sentence match intake running only on the operator's word and reading the hub's own repository?"
---

Questions to settle against the board entry as amended; each site is stated in full.
