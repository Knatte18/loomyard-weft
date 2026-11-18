```yaml
lens: forced-edits
outcome: reported
```

Forced-edits review of plan cards 1–5: callers, tests, comments, docs and registrations that the changes force.

**LOW — Card 5 (D4): two descriptions of the overview window are left incomplete.**
- `internal/battencli/overview.go:1` (file header) says the window draws "the dobby's lines (overviewdobby.go) above the orch lines of the beam and of every beam below it".
- `internal/battencli/overview.go:481` (`overviewCmd`'s `Short`) says "draw a beam's window: its dobby and its orchs".
- Once card 5's own-run line opens the frame, neither text names it. Card 5 updates only the `Long` help text, the `overviewSource` comment and the `frame` comment.
- Fix: the Intent says the file header and the `Short` text also name the own-run line. Listing the file self glyph beside its members would be refused by `redundant-file-target`, so the change belongs in the Intent prose.

**LOW — Card 4 (D3): one stale comment on `openOwnWindowLocked`.**
- `internal/reedengine#Engine.openOwnWindowLocked`'s doc (`internal/reedengine/spawn.go:252-258`) says it "opens a detached window for s named name".
- Card 4 changes which `name` its callers pass but does not list the method, so that comment can no longer state the title rule.
- The comment stays true as written ("named name"), so this is LOW. Card 4 needs no edit unless the writer wants the rule stated at the open site.

Tool calls made beyond my context:
1. Grep for `statusStrandCmd`, `statusSidecarPath`, `statusSidecarFile`, `status-strand.json`, `ensureRunsStrand` and `statusStrandAddSpec` repo-wide, plus `reedengine` importers in tests and the loomengine test package clause. Context lacked these callers outside the files already read.
2. `lym quarry glyphs` for `smoke_drivingsurface_test.go` and a grep of import-guard tests for `reedengine`. Context lacked the smoke test's span and any import guard.
3. Read `internal/cliwire/seam_enforcement_test.go` and ran `lym quarry resolve` on the cards' existing glyphs (`orchCLI.beamUp`, `placeApart`, `loomCLI.ensureStatusStrand`, `TestStatusStrandCmd`, `fakeStrands.ReplaceStrand`). Context lacked the guard's scope and resolve confirmation.
4. Grep for docs and comments naming `loom-status` or the status strand, production `Display.Title` users, and other copies of the `overview` help text. Context lacked the repo-wide set of descriptions the change could make wrong.
5. Grep of `internal/loomcli/checklist.go`, `start.go`, `cli.go`, `internal/statuspane` and `internal/loomengine/status.go` comments for overview-window or sidecar wording. Context lacked those files.
6. Grep of the `loomshed`, `shedengine`, `shedarbiter`, `reportengine` and `sheddriver` seam tests for transitive or `reedengine` bans. Needed to confirm that card 1's new `loomengine` → `reedengine`/`render`/`shell` imports trip no import guard. They don't: each guard checks only direct imports.
