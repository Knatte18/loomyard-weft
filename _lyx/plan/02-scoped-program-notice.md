# Card 2 — scoped-program-notice

**Edit:**
- `internal/creelcli#NewProgramClient`
- `internal/orchcli#postNotice` -> `func postNotice(location *lyxcwd.Location, slug, program, line string, orch func() (string, bool, error)) (bool, error)`
- `internal/orchcli#NotifyPrime`
- `internal/orchcli#NotifyParent`
- `internal/orchcli#NotifyFrom`
- `internal/orchcli#NotifyOrchAt`

**Create:**
- `plan:internal/creelcli#programClaim` -> `func programClaim(slug, program string) func(shortname string) (creelengine.Claim, error)`
- `plan:internal/creelcli#NewScopedProgramClient` -> `func NewScopedProgramClient(location *lyxcwd.Location, slug, program string) (*GoClient, error)`
- `plan:internal/orchcli#NotifyFromScoped` -> `func NotifyFromScoped(from *lyxcwd.Location, slug, program, line string) (posted bool, err error)`
- `plan:internal/creelcli#TestProgramClaim` -> `func TestProgramClaim(t *testing.T)`

**Uses:**
- `plan:internal/agentname#FormatScopedProgram`

**Intent:**
Give a program sender a beam scope end to end, so dobby's notices to the nearest live orch come from `ly:<slug>:DOBBY` (D1).
`programClaim` builds the claim a client signs with: `agentname.FormatScopedProgram(shortname, slug, program)`, which is the two-segment program for an empty slug; `NewProgramClient` becomes `newGoClient(location, programClaim("", program))`, and `NewScopedProgramClient` passes the slug.
`postNotice` takes the slug and builds its program client through `NewScopedProgramClient`; `NotifyPrime`, `NotifyParent`, `NotifyFrom` and `NotifyOrchAt` pass an empty slug, so every existing sender keeps its name.
`NotifyFromScoped` is `NotifyFrom` with the slug passed through: the walk to the nearest live orch starts at `from` itself.
`TestProgramClaim` (tier0, parallel) rows: an empty slug claims `ly:BATTEN`; slug `pay` claims `ly:pay:DOBBY`; a bad program name returns the agentname error wrapped as "the program name ... is not usable".

**ImpactSummary:** postNotice's four callers in orchcli pass an empty slug; creel's claim check already accepts the scoped form through ParseProgram.
