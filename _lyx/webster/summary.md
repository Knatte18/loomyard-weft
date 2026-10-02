# Agent names `<code>:<slug>:<role>`, renamed to shortname

This session ran the last four batches; the other sixteen were already `done` on resume.

- **17 agentname-shortname** (`ef3d5e971`): `internal/agentname` renamed its code segment to shortname (`ValidateShortname`, `StandaloneShortname`, `Name.Shortname`), and the callers in `fabricengine`, `fabriccli`, `reedengine` and `standalonegeom` followed.
- **18 fabric-shortname** (`1109d5efe`): the `.lyx-code` record became `.lyx-shortname`, `--code` became `--shortname`, and `lyx fabric code` became `lyx fabric shortname`.
  The engine symbols, envelope keys and `refusal-spec.md` follow.
- **19 reed-shortname** (`08fd95063`): `NameCode` became `NameShortname` in `reedengine`, `hubgeom` and `standalonegeom`, with the `reedcli` help texts and test names.
- **20 shortname-docs** (`8722cd627`): `CONSTRAINTS.md`, `docs/overview.md` and the `doc.go` files of `fabricengine`, `reedengine` and `hubgeom` now say shortname.

Batches 17 and 18 recorded informational scope-outside-plan warnings, all in files the renames had to touch; none was a deviation.
Batch 18 ran targeted integration tests only; the plan-level suite covered the rest.

The plan-level `## verify:` suite, run by the integration fork at HEAD `8722cd627`, reported `status: OK`.
