# Card 20 — shortname-docs

**Prosa:**
- `CONSTRAINTS.md`
- `docs/overview.md`
- `internal/fabricengine/doc.go`
- `internal/reedengine/doc.go`
- `internal/hubgeom/doc.go`

**Intent:**
Bring the docs to the "shortname" spelling cards 17–19 shipped, so no "code" spelling for the repo's short name is left anywhere in the repo.

- `CONSTRAINTS.md`'s Agent Name Invariant: the forms read `<shortname>:<role>` and `<shortname>:<slug>:<role>`, and the record bullet says the hub's shortname is recorded in `.lyx-shortname` beside `.lyx-warp`, a standalone run's derived shortname told by `internal/standalonegeom`.
- `docs/overview.md`: the `internal/agentname/` tree line reads `<shortname>:<slug>:<role>`;
  the fabric entry's CLI surface lists `shortname` in place of `code`, and its sentence says `clone` takes `--shortname` to record the repo's shortname as `.lyx-shortname` on weft:main, and `shortname [<shortname>]` prints or records it.
- `internal/fabricengine/doc.go`'s record paragraph: the hub's shortname, `.lyx-shortname`, `CommitShortname` in `shortnamecommit.go`, `shortnamebinding.go`, `lyx fabric clone --shortname <shortname>`, `lyx fabric shortname [<shortname>]`, and "a repo bound before shortnames existed".
- `internal/reedengine/doc.go`: an empty told shortname refuses, naming `lyx fabric shortname <shortname>`.
- `internal/hubgeom/doc.go`: it reads the hub's recorded shortname.
The occurrences of "code" meaning source code in these files stay.

**Verify:** ! git grep -nE 'lyx-code|fabric code|--code([^a-z]|$)|<code>:<|ValidateCode|StandaloneCode|NameCode|ReadCode|WriteCode|CommitCode|CodeFileName|hubforge\.TestCode|[Uu]ncoded|codebinding|codecommit|short code' -- . ':(exclude)_lyx' ':(exclude).lyx'

**Commit:** 20: docs: the repo code is now the shortname
