```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the F1 proposal, the reworded `ConfigDirs` sentence in `_lym/landing/summary.md`.

no findings

The proposed wording holds against the code:
- `configengine.Dirs` has `Records` (`internal/configengine/layers.go:22-29`). Its doc comment says "no layer reads it", which matches the proposed phrase "that no config layer reads".
- `ConfigDirs` sets `Records` from `RecordsWorktree(l)` (`internal/fabricengine/junctionnames.go:131`). So "tells the weft checkout" is accurate. Taking the weft checkout as the records checkout is the code's own pairing.
- The only reader of `Dirs.Records` outside `configengine` is `codexengine.go:129`. That is the beam's own code, not a call main's commits make. The hibernate call at `internal/hibernatecli/cli.go:101` and `:120` only passes the `Dirs` through to `hibernateconfig.LoadConfig`. The claim that this call is unaffected stands.
- The split into two sentences leaves the stencil claim unchanged, as the review asked.

Tool calls:
1. Ran `grep` for `.Records`/`Records:` readers under `internal`, and for the `type Dirs` block. My context showed `ConfigDirs` and the hibernate call, but not the `Dirs` doc comment or every reader of the field.
