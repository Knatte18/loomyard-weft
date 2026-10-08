# Card 6 — Retire the single-line status pipeline and `tokenvocab`

**Delete:**
- `internal/reedengine/status-line.md`
- `internal/reedengine/statuslinetemplate.go`
- `internal/reedengine/statusline.go`
- `internal/reedengine/statusline_test.go`
- `internal/reedcli/statusline.go`
- `internal/reedcli/statusline_test.go`
- `internal/tokenvocab/doc.go`
- `internal/tokenvocab/tokenvocab.go`
- `internal/tokenvocab/render.go`
- `internal/tokenvocab/tokenvocab_test.go`
- `internal/tokenvocab/leaf_enforcement_test.go`

**Edit:**
- `cmd/lyx/testdata/test-budget.yaml`
- `internal/reedengine/config.go`
- `internal/reedengine/template_posix.yaml`
- `internal/reedengine/template_windows.yaml`
- `internal/reedengine/template.go`
- `internal/reedengine/lifecycle.go`
- `internal/reedengine/lifecycle_test.go`
- `internal/reedengine/config_test.go`
- `internal/reedengine/geometry.go`
- `internal/reedengine/lock_test.go`
- `internal/reedengine/ensuresession_integration_test.go`
- `internal/reedengine/mouse_boot_integration_test.go`
- `internal/reedengine/contract_integration_test.go`
- `internal/reedengine/doc.go`
- `internal/hubgeom/hubgeom.go`
- `internal/hubgeom/hubgeom_test.go`
- `internal/standalonegeom/reedgeom.go`
- `internal/standalonegeom/standalonegeom_test.go`
- `internal/reedcli/cli.go`
- `internal/reedcli/smoke_statuslineseed_test.go`
- `cmd/lyx/stencilseedgate_test.go`
- `internal/clihelp/annotations_test.go`
- `internal/configsync/configsync_test.go`
- `internal/configreg/fill_test.go`
- `internal/githubclient/leaf_enforcement_test.go`
- `internal/standalonestate/leaf_enforcement_test.go`
- `internal/pattern/leaf_enforcement_test.go`
- `internal/buildinfo/leaf_enforcement_test.go`
- `.gitattributes`
- `PATTERN.md`
- `pattern/PATTERN-leaf-packages.md`
- `pattern/PATTERN-told-geometry.md`
- `docs/overview.md`
- `docs/shared-libs/yamlengine.md`

**Uses:**
- `internal/reedcli/switch.go`

**Intent:**
Remove the single-line status-text pipeline the bar of card 5 replaced (D16), so no config key or verb survives whose output is never displayed.

What goes: the embedded `status-line.md` template and its accessor, `Engine.StatusLineText` and `Engine.ValidateStatusLine` with the boot validation call in `ensureServerAndSessionLocked` (and its long comment), the `lyx reed statusline` verb and its registration in `cli.go`, `Config.StatusLine`/`StatusLineConfig` and the `status_line` block of both templates, the whole `internal/tokenvocab` package (its only consumer was this pipeline), its `.gitattributes` line, and `Geometry.RepoName`, which only the status text read, together with its two tellers' assignments (`hubgeom.go`, `standalonegeom/reedgeom.go`) and the test literals and assertions that set or read it, the `tmux`-tier `Geometry` literals in `ensuresession_integration_test.go`, `mouse_boot_integration_test.go` and `contract_integration_test.go` included.
`geometry.go`'s comments for `WorktreeName` and `HubPath` stop calling them status-line tokens and state what still reads them (the empty-shortname refusal and the told-identity error messages).

What stays loadable: a materialized `reed.yaml` still carrying `status_line` keeps loading with the key ignored; `config_test.go` gains that row (`PATTERN-config-strictness`).
`lifecycle_test.go`'s `TestUp_BootValidation` drops its bad-template row.

The stencil-seed annotation tests lose their only annotated command, so they move to `lyx reed switch` (card 4), which carries `clihelp.SkipStencilSeedAnnotation`: `cmd/lyx/stencilseedgate_test.go`'s lookup and row, `internal/clihelp/annotations_test.go`'s comment, and `internal/reedcli/smoke_statuslineseed_test.go`, whose run of the built binary now runs `lyx reed switch` with flags that fail fast (a socket path with nothing listening) and still asserts an empty stderr from the stencilstore emitters; its header comment says so.

Tests that name the retired key follow the template: `configsync_test.go`'s stale-header row expects `selvage.height_rows` added (and no `status_line.template`), and `configreg/fill_test.go`'s "reed dropped whole mapping" row drops `segment_colors` instead of `status_line`, which also pins that a missing color block fills from the template.
The four leaf-enforcement tests whose header comments cite tokenvocab's test as their model cite modelspec's instead.
`test-budget.yaml` drops the `internal/tokenvocab` row and sets the `internal/reedengine` and `internal/reedcli` rows to each package's top-level test count after this card.

PATTERN: `PATTERN.md`'s `PATTERN-leaf-packages` line drops `tokenvocab`, `pattern/PATTERN-leaf-packages.md` drops its section, and `pattern/PATTERN-told-geometry.md` drops it from its package list.
`docs/overview.md` drops `tokenvocab` from the repository tree, the shared-infrastructure list, the "shared leaf" bullet and the package-documentation list, and `statusline` from the reed verb list.
`docs/shared-libs/yamlengine.md`'s example of a whole missing mapping names `segment_colors` instead of `status_line`.
`reedengine` `doc.go` drops every statusline/tokenvocab mention; the measurement records under `docs/benchmarks/` stay as they are.

Acceptance: `lyx reed statusline` no longer exists, and `go build ./...` shows nothing imports `internal/tokenvocab`.

**ImpactSummary:** removes one verb, one config key, one Geometry field and one package; three tellers/tests updated, behavior of live sessions unchanged after card 5.
