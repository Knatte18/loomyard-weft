# Card 7 — owner-seam-wiring

**Edit:**
- `internal/pairteardown#SeamOptions`
- `internal/pairteardown#HubSeamOptions`
- `internal/pairteardown#Teardown.OriginSeams`
- `internal/pairteardown#TestTeardown_OriginSeams`
- `internal/pairteardown/doc.go`

**Create:**
- `plan:internal/pairteardown#OwnerReader` -> `func OwnerReader(machine func() (string, error), parentOf func(hubPath, slug string) (string, error)) fabricengine.OwnerReader`

**Uses:**
- `plan:internal/machinename#Name`
- `plan:internal/boardengine#ParseOwners`
- `plan:internal/boardengine#Owners.Effective`
- `plan:internal/boardengine#OwnedElsewhere`
- `plan:internal/fabricengine#ParentSlug`

**Intent:**
Every reconcile call site builds its seams through `pairteardown`, so the owner reader is built here once (D4).
`OwnerReader` builds the one `fabricengine.OwnerReader`: it reads the machine name (an error, the empty-hostname one included, is returned, so no owner is compared with an empty name and the reconcile asking it fails naming the cause, D1), parses the bytes with `boardengine.ParseOwners`, takes the slug's effective owner with `parentOf` as the parent resolver, and reports it with `boardengine.OwnedElsewhere`; a parse or `parentOf` error is returned.
`SeamOptions` gains `Machine func() (string, error)`, this machine's name, and `ParentOf func(slug string) (string, error)`; `HubSeamOptions` fills `Machine` with `machinename.Name` and `ParentOf` with `fabricengine.ParentSlug` over the prime's location, and a nil `Machine` builds no `OwnerOf`, so a call site that sets none keeps today's verdicts.
`Teardown.OriginSeams` fills `OwnerOf` with `OwnerReader` over the options' `Machine` and `ParentOf`.
`doc.go` names the owner seam beside the live-pair, parent-lock and teardown seams.
`TestTeardown_OriginSeams` gains rows over `OwnerOf` with a fixed `Machine` and a fake `ParentOf`: an entry recorded for this machine is owned here, one recorded for another is owned elsewhere with that owner, an unowned entry under another machine's prime owner is owned elsewhere, an unowned task whose fake parent is a pair owned elsewhere is owned elsewhere, a hub with no prime owner is owned here, a `Machine` error is returned, a `ParentOf` error is returned, undecodable bytes return the error, and a nil `Machine` builds a nil `OwnerOf`.

**ImpactSummary:** one seam builder in pairteardown; every reconcile caller (batten, loom landing, hibernate, orch, cleanup) now gets owner verdicts through `HubSeamOptions`.

**Commit:** 7: pairteardown: build the reconcile's owner reader over the board
