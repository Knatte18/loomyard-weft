# Card 16 — naming-docs

**Prosa:**
- `docs/overview.md`
- `internal/reedengine/doc.go`
- `internal/fabricengine/doc.go`
- `internal/shedrun/doc.go`

**Uses:**
- `CONSTRAINTS.md`

**Intent:**
Bring the module documentation up to the shipped naming, pointing at the Agent Name Invariant in `CONSTRAINTS.md` rather than restating it.

- `docs/overview.md`: the package tree gains `internal/agentname/` with a one-line purpose;
  the fabric module entry lists the `code` verb in its CLI surface and says `clone` takes `--code` and records `.lyx-code` on weft:main;
  the reed module entry lists the `list` verb and says strands carry a hierarchical full name, mirrored into pane titles and Claude session names and exported as `LYX_STRAND_NAME`/`LYX_PARENT`, with the watchdog repairing drift.
- `internal/reedengine/doc.go`: how a strand's full name is formed (the told prefix, `-N` numbering, explicit names refusing when held), that the name is a birth attribute and the only lookup key, the env exports, the pane-title mirror with `allow-set-title off`, the watchdog's name-repair tick and the `SessionNamer` seam, `lyx reed list`, and the retired `strand_name` key.
- `internal/fabricengine/doc.go`: the `.lyx-code` record beside `.lyx-warp`, the clone rules for `--code`, and `lyx fabric code`.
- `internal/shedrun/doc.go`: the seed's optional, write-once `parent` field and who writes it.
Nothing under `manifest/` changes: it is being retired, and the bolt `creel` note already records that this grammar supersedes reed-mailbox's addressing.

**Commit:** 16: docs: agent names, the short code and reed list
