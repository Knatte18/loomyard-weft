Lens fix-consistency, one finding:

- **LOW** — `contracts/stencils/loom/loom-template-arbiter.md` and the three other banners were fixed, but their twin at `contracts/stencils/loom/loom-template-discussion-advisor.md:6` still reads `{{.parent_directive}}, {{.edit_directive}} and {{.pattern_directive}} are the shared directives.`
  - Evidence: the same grep that found the four fixed banners returns this line. It still describes `edit_directive` as shared, though no shared copy exists any more.
  - Fix: reword the line so `edit_directive` is the edit directive of the launch engine.

Checked and consistent:
- No `driverPrompt` reference is left in Go, markdown or yaml outside the review and run logs after the `RenderDriverPrompt` rename.
- The `TidyShuttle` signature change shows no leftover caller or doc reference in `dobbyengine` or `beamcli` docs.
- The `docs/research/.../report.md` mention of "the shared edit directive" is a dated measurement report, so I left it alone.

Tool calls:
1. One `grep` batch over the Go, markdown and yaml files, covering stale `driverPrompt`, "shared edit directive", `EngineName` and "stay unexported" mentions. My context held only the edited files, not the rest of the repository.
