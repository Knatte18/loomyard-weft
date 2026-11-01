```yaml
lens: code-affected-set
outcome: reported
```

Task: list what the diff's change to the board's rendered shape forces outside the lines it touches, per change.

**Per change, the set found**

1. `projectBody` now adds an own-entries heading (`No project` or `Own entries`) when a project has open sub-projects. This changes the shape of every rendered README and project file.
   - **Readers of the rendered output.** A repo-wide grep for `## Prime`, `### Tasks (`, `## Tasks (`, `#### Tasks (` and `project-*.md` outside `boardengine/render*.go` and `doc.go` found no assertion on heading shape. Only these names turned up:
     - `fabricengine/replay_integration_test.go:563` has the fixture `"project-x.md": "# P1 X\n"`, a file name and header only.
     - `boardengine/replay_test.go:638-677` checks file-name classification only.
     - `boardengine/rename_test.go:137-144` checks that the file names the new slug and not the old one.
   - **Tests.** The `render_test.go` goldens are updated. They cover `No project` at `###`, `Own entries` at `####` and `##`, and the level-six cap. Covered.
   - **Docs.**
     - `docs/overview.md:364` is updated. Covered.
     - `doc.go:172-186` is updated. This covers the own-entries paragraph, the project-file `##`/`###` rule and the counted-heading exclusion. Covered.
     - `render.go:1-8` header and the `projectBody` comment are updated. Covered.
     - No other doc, stencil, `PATTERN.md` or `pattern/` file describes the rendered shape. The grep of `contracts`, `docs`, `pattern` and `PATTERN.md` found only `docs/overview.md`. Covered.
   - **Anchors and links.** The heading text is plain (`heading()` with no link or count), and nothing else references headings by anchor. Covered.
2. `entrySections` is now called at `level+2` when sub-projects exist. It is a private function, and its only caller is `projectBody`. Covered.
3. **PATTERN and declared changes.** The diff touches no `PATTERN.md` or `pattern/` file, and the directive declares no PATTERN changes. Nothing is owed.
4. **Commit message.** It has no `Pattern-Departure:` trailer, and none is needed because no pattern is departed from. Covered.

**Findings**

None.

One observation, not a finding. `doc.go:172` says every project renders "through one project section" and does not mention the own-entries heading. Lines 176-182 do, so nothing is left stale.

**Tool calls (3)**
1. A repo-wide grep for rendered-shape phrases, to find files that describe or assert the board's shape. My context lacked a search outside `boardengine`.
2. A grep of non-`boardengine` files for rendered headings and `project-*.md`, and of `doc.go` and `overview.md` for the new wording. My context lacked these callers.
3. Reads of `render.go:100-128` and `doc.go:168-176`, plus a grep of the md docs for a Prime/README description. My context lacked the `Render` caller and the surrounding doc text.
