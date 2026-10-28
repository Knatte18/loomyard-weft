---
round: 1
focus:
  - "internal/battenshed/template.yaml is in the diff but no scope bullet or decision names it: settle whether the board entry or the record licenses the change."
  - "Docs landing: the record's scope lists the package docs of the touched packages and docs/overview.md's Seed-IDE line; only boardengine/doc.go and docs/overview.md appear in the diff stat. Settle whether the doc.go of internal/vscode, internal/ideengine, internal/idecli, internal/beamcli and internal/fabricengine needed an update and got none."
  - "internal/vscode/config.go: the settings writer rewrites only the keys lym owns in an existing settings.json (D4). Settle that other keys survive, that an undecodable JSONC file is left byte-identical with a warning, and that no status-bar, border or tab colour key is written."
  - "internal/vscode/color.go: the luminance rule (D5) uses WCAG relative luminance with the 0.179 threshold, including the sRGB linearisation. Settle the arithmetic, the inactive-text alpha, and that the palette's green is #1b5e20 as mainColor and palette[0]."
  - "internal/ideengine/spawn.go: activity-bar picking for a task (D3) avoids sibling tasks' colours, avoids the project colour, keeps the task's own current colour when still allowed and free, and degrades when the palette runs out. Settle that the parent is read only through hubgeom.ParentOf and that a beam with no recorded colour is read, not recorded."
  - "internal/boardengine ownerset.go, task.go, migrate.go, replay.go: beam_color is set only by the owner setter, only when a Beam request records the beam and the entry has no colour, picked inside the board write lock, and dropped by normalizeTask from a non-beam entry. Settle round-trip through load, save and replay."
  - "internal/beamcli/create.go: the owner step re-runs for a beam recorded here with no beam_color (D2), the pick excludes green and other open beams' colours, and an already-coloured beam is left alone."
  - "internal/idecli/cli.go: with no slug the verb seeds Location.WorktreeName, the prime included; a slug behaves as before and the prime's name is accepted as a slug; the help names the prime; lym ide open keeps its required slug and fabric clone stays IDE-free."
  - "internal/fabricengine/hubworkspace.go: a beam's .code-workspace splices the beam's own .vscode/settings.json when it exists and the prime's otherwise. Settle the failure path for an unreadable or invalid beam settings file, and that window.title in the workspace file matches the folder's."
  - "Window title (D6): the hub shortname upper-cased, a colon, the worktree's slug; a task under a beam names only its own slug; a hub with no recorded shortname falls back to the slug without failing a seed."
  - "Test economy across the diff: new tests in internal/ideengine/look_test.go, internal/vscode/color_test.go and config_test.go against existing tests that already cover the same behaviour, and any behaviour the diff adds with no test."
---

Seed focus drawn from the board entry's design and the record's decisions D1 to D8, against the diff stat of the review range.
