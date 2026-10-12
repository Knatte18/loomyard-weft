# Card 5 — loom-arbiter-records

**Create:**
- `internal/loomengine/arbiter.go`
- `internal/loomengine/arbiter_test.go`
- `plan:internal/loomengine#ArbiterRole` -> `const ArbiterRole = "arbiter"`
- `plan:internal/loomengine#IsArbiterStrand` -> `func IsArbiterStrand(name string) bool`
- `plan:internal/loomengine#ArbiterStopKey` -> `type ArbiterStopKey struct`
- `plan:internal/loomengine#StopKeyOf` -> `func StopKeyOf(st shedengine.Status) ArbiterStopKey`
- `plan:internal/loomengine#ArbiterAttempt` -> `type ArbiterAttempt struct`
- `plan:internal/loomengine#WriteArbiterAttempt` -> `func WriteArbiterAttempt(dir string, a ArbiterAttempt) error`
- `plan:internal/loomengine#ListArbiterAttempts` -> `func ListArbiterAttempts(arbiterDir string) ([]ArbiterAttempt, error)`
- `plan:internal/loomengine#ReadArbiterAttempt` -> `func ReadArbiterAttempt(dir string) (ArbiterAttempt, error)`
- `plan:internal/loomengine#ArbiterVerdict` -> `type ArbiterVerdict struct`
- `plan:internal/loomengine#ReadArbiterVerdict` -> `func ReadArbiterVerdict(path string) (ArbiterVerdict, bool, error)`
- `plan:internal/loomengine#ArbiterPromptPath` -> `func ArbiterPromptPath(dir string) string`
- `plan:internal/loomengine#ArbiterVerdictPath` -> `func ArbiterVerdictPath(dir string) string`
- `plan:internal/loomengine#ArbiterEnvelopePath` -> `func ArbiterEnvelopePath(dir string) string`
- `plan:internal/loomengine#ArbiterEnvelope` -> `type ArbiterEnvelope struct`
- `plan:internal/loomengine#WriteArbiterEnvelope` -> `func WriteArbiterEnvelope(path string, e ArbiterEnvelope) error`
- `plan:internal/loomengine#ReadArbiterEnvelope` -> `func ReadArbiterEnvelope(path string) (ArbiterEnvelope, bool, error)`
- `plan:internal/loomengine#MarkArbiterApplied` -> `func MarkArbiterApplied(dir string) error`
- `plan:internal/loomengine#ArbiterApplied` -> `func ArbiterApplied(dir string) (bool, error)`
- `plan:internal/loomengine#ArbiterMarker` -> `type ArbiterMarker struct`
- `plan:internal/loomengine#WriteArbiterMarker` -> `func WriteArbiterMarker(path string, m ArbiterMarker) error`
- `plan:internal/loomengine#ReadArbiterMarker` -> `func ReadArbiterMarker(path string) (ArbiterMarker, bool, error)`
- `plan:internal/loomengine#RemoveArbiterMarker` -> `func RemoveArbiterMarker(path string) error`

**Edit:**
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Declare the arbiter's records and their readers and writers in one new file, `internal/loomengine/arbiter.go`, so `internal/loomcli` writes them and `internal/battencli` reads the marker without importing another CLI package (the precedent `LoomDriverStrandName` set).
Every function takes the directory or path it works on; `internal/shedrun` (card 1) declares where they live.

- `ArbiterRole` is the arbiter's agent-name role (D11); `IsArbiterStrand` reports a name that `agentname.Parse` accepts and whose role `agentname.MatchesRole` matches to it, so a numbered `arbiter-2` counts.
- `ArbiterStopKey` is the stop an attempt arbitrates (D1): `CurrentProducer`, `HistoryLength` and `State`; `StopKeyOf` reads it off a status.
- `ArbiterAttempt` is `attempt.json` in an attempt directory: `Number`, `StopKey`, `PauseBefore`, `PauseAfter`, `SpawnedAt`, `Deadline`, `StrandGUID`, `EnvelopePath`, `ReportPath`, plus `Dir`, filled by the readers and never serialized.
  `WriteArbiterAttempt` creates the directory and writes the file; `ReadArbiterAttempt` reads one; `ListArbiterAttempts` reads every subdirectory of the arbiter directory, sorted by `Number`, returns none for an absent directory, and fails naming a subdirectory with no readable `attempt.json`, since the verb writes that file before anything else and its count is the spawn budget (D7).
- `ArbiterVerdict` is the agent-written `verdict.yaml` (D3): `Verdict` (`resume` or `hand-on`), `Reason`, `Verbs []string`, `Goto` and `Question`.
  `ReadArbiterVerdict` returns found false for an absent file, and an error for a file that does not parse, an unknown verdict word, an empty reason, a `hand-on` without a question, or a `goto` on a `hand-on`.
- `ArbiterPromptPath`, `ArbiterVerdictPath` and `ArbiterEnvelopePath` name `prompt.md`, `verdict.yaml` and `envelope.json` in an attempt directory.
- `ArbiterEnvelope` is `envelope.json`, the verb's validated result for an attempt: `Verdict`, `Reason`, `Question`, `Goto`, `Verbs`, `Cause` (empty for the agent's own verdict), `Attempt`, `AttemptDir`, `VerdictPath` and `Relaunch`.
  `WriteArbiterEnvelope` writes it, and `ReadArbiterEnvelope` returns found false for an absent file and an error for one that does not parse.
- `MarkArbiterApplied` writes the `applied` mark file in an attempt directory, and `ArbiterApplied` reports whether it exists.
- `ArbiterMarker` is the in-flight marker (D9): `Attempt`, `StrandGUID` and `Deadline`.
  `WriteArbiterMarker` creates its directory and writes it, `ReadArbiterMarker` returns found false for an absent file, and `RemoveArbiterMarker` ignores an absent one.

`arbiter_test.go` holds one table-driven test per reader: attempt write, read and list (absent directory, ordering, a subdirectory without `attempt.json` refused), verdict parse (each refusal above and both valid shapes), applied mark, marker round trip and absence, envelope round trip, absence and an unparseable file, and `IsArbiterStrand` (plain, numbered, another role, an unparsable name).
Raise `internal/loomengine`'s row in the test budget by the tests added.

**ImpactSummary:** a new file of records; only the test budget row is an existing file changed.
