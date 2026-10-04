# PATTERN.md replaces CONSTRAINTS.md; manifest/ retired

Loomyard's invariants move from `CONSTRAINTS.md` into `PATTERN.md` at the repo root.
`PATTERN.md` is a thin overview of one-line `PATTERN-<name>` entries grouped under topic headings.
An entry that needs more than a line links a background file, `pattern/PATTERN-<name>.md`.
Until now no loomyard run received the PATTERN directive, because the file never existed.
Every lyx agent that touches design, code or review now gets the overview inlined in its prompt.

The same change retires `manifest/` and folds in five repo conventions.
No issue is closed by this change.

## What changes

- **PATTERN files.**
  `CONSTRAINTS.md` is deleted.
  Each of its sections is now an entry, with near-duplicate sections merged.
  Every file that cited `CONSTRAINTS.md` (`CLAUDE.md`, stencils, specs, `docs/`, code comments, `doc.go` files, the ly-drive skill) cites the entry name instead.
- **`internal/pattern`.**
  - `pattern.Directive` reads `PATTERN.md` from the warp worktree root and fills the new `{{.pattern_overview}}` marker in each directive stencil.
  - Background files stay a relative `pattern/` pointer.
  - Two roles are added, `RoleDesigner` (Discussion-Write) and `RoleJudge` (Bouncer judge), each with its own stencil.
  - A pure format checker reports a bad or duplicate name, a multi-line entry, a line over 320 characters, an overview over 16 KiB, an empty overview, an overview with no entry, an unresolved link or one that points at another entry's file, and an orphan background file.
  - A real-file test runs the checker on the module's `PATTERN.md` and never skips.
- **`fabricengine`.**
  The PATTERN-residue report is removed from the pull result and from the `lyx fabric pull` envelope (`pattern_residue`).
  It scanned weft commits for paths that no longer live in the weft.
- **`manifest/`.**
  Surviving Someday items became tier-3 board notes.
  Designs of items already on the board were appended to those entries.
  Reference content moved into the package docs of `orchcli`, `treadleengine`, `fabricengine`, `fslink`, `websterengine`, `burlerengine` and `boardengine`.
  Inbound links are fixed and `docs/overview.md` and `CLAUDE.md` now say that an unbuilt design lives in its board entry and a built one in its package's `doc.go`.
- **Fabric vocabulary.**
  `TestEnforcement_FabricVocabulary` also scans test files outside the owner set and the `.md`, `.yaml` and `.go` files under `contracts/`, `docs/`, `plugins/`, `crucible/` and `cmd/`, plus `CLAUDE.md` and `README.md`.
  Existing hits are reworded.
- **Model spec.**
  The short form (`model[effort]`) is canonical in `llm-model-spec.md`, template comments link to that spec, and the `landingshed` fixtures are short-form.
  The parser is unchanged.
- **Role constants.**
  Every spawn site takes its role from a constant in its own module, and `cmd/lyx/spawnrole_test.go` fails on an inline literal role.
- **ide hub workspace.**
  - The prime path is built through `WorktreePath`.
  - A failed prime resolution logs at WARN only for a prime spawn.
  - Removing the last pair keeps `_portals/<anchor>`, and the workspace lists existing folders only.
  - A spliced `./` or `../` setting value logs a warning naming the key.
- **`shedrun.Seed.Parent`** and its fallback are removed.
  A seed file that still carries `parent` decodes, and the key is ignored.

## Design decisions to review

- PATTERN is tracked warp code that lands through the ordinary merge, not weft.
  Its text carries no wording about agents or the dev workflow; that stays in the stencils and `internal/pattern`.
  This is review-enforced, not tested.
- Whitespace-only `PATTERN.md` yields no directive instead of an empty one, since `stencil.Fill` rejects an empty required marker.
  Malformed content is inlined as is; the checker test, not the runtime, catches it.
- The overview is inlined on every directive prompt, so prompts grow by up to the 16 KiB cap.
  Many entry lines run past 200 characters but stay under the 320 cap.
- The vocabulary-scan allowlist is a closed list in the test, each path with a reason line.
- Removing `Seed.Parent` assumes no run seeded before `aa67fd1d7` is still live.
  A live one would behave like a new run without an origin `parent_worktree`.

## To check by hand

- Run `lyx board list` and confirm the new notes from the `manifest/` retirement exist and read correctly; the board writes were upserts and are not undone if this change is rejected.
- Confirm no run seeded before `aa67fd1d7` is live in the hub.
- The edited ly-drive skill ships only through `update-plugins.sh`; it was not run.
- `lyx board get` warns that the board copies of the stencils have drifted from the worktree source; a production deploy is needed to refresh them.
- The plan-level verify gate had not reported when the run record was written, so confirm `go test ./...` and `go vet ./...` pass.
