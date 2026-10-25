---
round: 1
focus:
  - "Amendment scope: the Parent-Align merge of afd030a47 (beam window, `orchcli.HubOrchs`, `orchcli.PrimePaths`, moved `DobbyRunCells`). Check that the merged code resolves each worktree through `lyxcwd.ResolveWorktree` and reads that pair's own marker, as the decision record's anchor decisions (D2, D3, D7) require."
  - "Check the merged parent code for any `lyxcwd.Location` literal or hand-built path the anchor change forbids in production code; verify the claim in the fixer report that the only new literal is in `overview_test.go`."
  - "Check that the text-only overlaps (`docs/overview.md`, `internal/beamcli/cli.go`, `internal/beamcli/doc.go`) keep both the parent's beam-window text and the task's anchor changes, with no dropped or contradictory sentence."
  - "Check that the fixer report's claims about what it changed under `## Fixes` are borne out by the diff since the settle, and that nothing outside the amendment is touched."
---

Seed focus for an amendment round: the review input is the Parent-Align fixer report, so the question is whether the alignment keeps the merged tree consistent with the decision record.
