# Card 11 — report-replies

**Create:**
- `plan:internal/reportengine#RoleReplies` -> `type RoleReplies struct`
- `plan:internal/reportengine#ComputeReplies` -> `func ComputeReplies(entries []creelengine.LogEntry) []RoleReplies`
- `plan:internal/reportengine#TestComputeReplies` -> `func TestComputeReplies(t *testing.T)`
- `plan:internal/reportcli#repliesCommand` -> `func repliesCommand() *cobra.Command`
- `plan:internal/reportcli#creelLog` -> `var creelLog = readCreelLog`
- `plan:internal/reportcli#readCreelLog` -> `func readCreelLog(location *lymcwd.Location) ([]creelengine.LogEntry, error)`
- `plan:internal/reportcli#TestRepliesVerb` -> `func TestRepliesVerb(t *testing.T)`

**Edit:**
- `internal/reportcli#Command`
- `internal/reportcli/doc.go`
- `internal/reportengine/doc.go`
- `docs/overview.md`

**Extends:**
- `internal/reportengine#allowedImports`

**Uses:**
- `internal/reportcli#resolvePrime`
- `internal/creelcli#NewProgramClient`
- `internal/creelcli#GoClient.Log`
- `internal/creelengine#LogEntry`
- `internal/agentname#Parse`
- `internal/agentname#ParseProgram`

**Intent:**
`lym report replies` counts, per receiver role, the expected replies, the replies, the chases and the unanswered messages over the creel journal's seven-day retention (D12), so the stencils of roles that often need chasing can be tuned.

`ComputeReplies` is pure over log entries and groups by the receiver's role: a strand's `agentname.Parse(...).Role`, a program's upper-case name from `agentname.ParseProgram`.
Per role, `RoleReplies` counts `Expected` (a `posted` entry with `ExpectReply`, by its `To`), `Replied` (a `posted` reply entry whose `ReplyTo` names a message the entries show as marked, by the reply's sender, the holder that answered), `Chased` (a `chased` entry, by its `To`) and `Unanswered` (an `unanswered` entry, by its `To`).
A reply whose `ReplyTo` names no marked message counts nowhere.
Roles come out in name order.
`allowedImports` in `internal/reportengine/seam_enforcement_test.go` gains `github.com/Knatte18/loomyard/internal/creelengine` and `github.com/Knatte18/loomyard/internal/agentname`, the two packages `ComputeReplies` imports, so `TestToldGeometryInvariant_AllowlistOnly` passes.

`repliesCommand` is the verb, beside `convergence` and `timing`: it resolves the prime through `resolvePrime`, which refuses any other worktree naming `lym report replies`, reads the journal through the package seam `creelLog`, whose production value `readCreelLog` calls `creelcli.NewProgramClient(location, "REPORT")`'s `Log`, and prints one `{"ok":true}` envelope holding `roles`; it takes no flags, writes nothing and changes nothing.
Bound: the program claim is accepted from the operator's terminal or an orch pane, the places the report verbs run from, and creel refuses it from another agent's pane, naming the claim.
`Command` registers it and its Long help names it; `internal/reportcli/doc.go` describes the verb and its source; `internal/reportengine/doc.go`'s opening, which names convergence and timing and says every figure is read from git objects, is reworded to cover `ComputeReplies` over creel log entries, and its body gains the computation; `docs/overview.md`'s report row names it.

`TestRepliesVerb` (`//go:build tier1`) drives the verb through `RunCLIIn` over a `hubforge` hub: run from a pair it is refused with exit 1 naming `lym report replies` and the prime, and run from the prime with `creelLog` replaced by a fake answering a fixed entry list it prints an ok envelope whose `roles` are that list's counts.
It replaces a process-global seam, so it does not run in parallel; it is a test of its own because `TestReportVerbs_RefuseOutsideThePrimeThenReportTheSeededRun`, which drives the other verbs over the same kind of hub, runs in parallel.

`TestComputeReplies` (`//go:build tier0`) is table-driven over a log-entry fixture: marked posts to a driver and an orch strand and to a program; a reply to a marked message; a reply to an unmarked message, which counts as no reply; two `chased` entries and one `unanswered` entry; and an empty journal, which answers no roles.

**ImpactSummary:** A new read-only report verb and a pure count over creel log entries; the existing report verbs are unchanged.
