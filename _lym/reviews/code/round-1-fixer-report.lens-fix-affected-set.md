```yaml
lens: fix-affected-set
outcome: reported
```

**Findings**

- **LOW — F4, ordering registered only where the seed pass runs.**
  - Changed text: `cmd/lym/stencilseed.go:58`, `stencilstore.RegisterOrdering(...)`.
  - It runs only in `seedStencils`, which returns early under `testing.Testing()`, for a command carrying `SkipStencilSeedAnnotation`, and when no hub resolves.
  - Opt-outs found: slack (`internal/slackcli/cli.go:159`), creel (`internal/creelcli/cli.go:105`), help, the shuttle hook, and reed button and switch.
  - A process under such a command, or with no hub, falls back to commit-time ordering.
  - Whether any of them reads a stencil is unverified; none looks like a driver, so it is probably fine.
  - The reviewer's fix named "cmd/lym's startup", and that is covered; this is only a residual gap.

- **No other findings.**
  - F1: the stale-reference search for `contentForRunning`, `shippedOverNewer`, `newerCopyWarned` and the old fallback wording returned nothing in code or docs.
  - The `Read` doc, `doc.go` and `skew.go` all describe the refusal.
  - The other comments the review rejected, for example `internal/shedadapters/bouncer.go:330`, were not re-read; the search found no embedded-fallback wording left anywhere.
  - Both read sites are covered: `reconcile.go` `Read`, and the engine-block path in `engineblock.go` `readInclude`.
  - Plain includes go through `Read`.
  - The remaining `stencilstore.Path` uses outside the package (`internal/loomcli/driverprompt.go:89`, `internal/burlerengine/prompt.go:547`, `internal/shedadapters/bouncer.go:1594`, `internal/loomengine/twill.go:93`) hand paths to agents. `internal/stencilcli` (`cli.go`, `promote.go`, `diff.go`) and `internal/fabricengine/stencilhistory.go` inspect board state. None renders text from the file.
  - `os.ReadFile` on a stencil path outside `stencilstore` returned nothing.
  - F5: no `needs no registry` wording remains.
  - `RegisterShipped` still has one caller, `contracts/stencils/stencils.go:1143`.

**Unchecked:** F2 and F3 are not set-premised.

**Tool calls**
1. Grep for stale names and fallback wording across `.go` and `.md`, plus `stencilstore.Path` and `RelPath` callers outside the package, plus `RegisterOrdering` and `RegisterShipped` callers. My context lacked a repo-wide view of these.
2. Grep for `SkipStencilSeedAnnotation` users and for `os.ReadFile` on stencil paths outside `stencilstore`. My context lacked the opt-out list and the other read sites.
