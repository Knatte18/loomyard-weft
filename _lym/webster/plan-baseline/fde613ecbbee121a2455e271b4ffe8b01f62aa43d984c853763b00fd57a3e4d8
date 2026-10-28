# Card 16 — agent-messaging-scan

**Create:**
- `cmd/lym/agentmessaging_test.go`
- `cmd/lym#engineToolAllowed`
- `cmd/lym#engineToolHits`
- `cmd/lym#goLiteralHits`
- `cmd/lym#testdataPromptRoots`
- `cmd/lym#TestAgentMessaging_NoEngineTool`

**Edit:**
- `PATTERN.md`

**Uses:**
- `internal/testkit/scankit#Walk`
- `internal/testkit/scankit#RequireFloor`

**Intent:**
A scan keeps `SendMessage` out (D13), and `PATTERN.md` gains the entry it enforces.
`cmd/lym/agentmessaging_test.go` is `//go:build tier0`, package `main`, beside `removedverbs_test.go`, whose shape it follows.

`engineToolHits` returns one `line N: SendMessage` entry per line of content naming `SendMessage`, skipping a line the allowlist admits for that file.
`engineToolAllowed` maps a repository-relative stencil path to the one line text admitted there: `contracts/stencils/webster/webster-template-master.md` admits the line holding "to that batch's fork by `SendMessage`, addressed by the agent id its completion notification carried", the in-session fork addressing of webster's Master, whose spec sets `ForkSubagents` (`internal/websterengine/runlevel.go`).
A new allowlist row is a reviewed change to the table, made only for a stencil whose role's spec sets `ForkSubagents`; the scan cannot read the spec from a stencil, so review holds that half.

`TestAgentMessaging_NoEngineTool` has two subtests.
`FixtureRows`, table-driven: over `engineToolHits`, a planted stencil line is a hit, the allowlisted line in the webster Master stencil is no hit, the same line in another stencil path is a hit and clean content has none; over `goLiteralHits`, a Go source whose string literal names the tool is a hit, one naming it only in a comment is none; and over `testdataPromptRoots`, a planted `internal/x/testdata` directory under a temporary root is listed.
`Tree` scans three surfaces from the module root with `scankit.Walk`: every file under `contracts/stencils/` but its `.go` files (`Filter: scankit.All`), whose non-test Go the Go surface reads; every `.md` prompt fixture in a `testdata` directory under `internal/`, whose directories `testdataPromptRoots` lists first with a `filepath.WalkDir` over `internal/` and passes as `Roots`, since `scankit` skips a `testdata` directory below a root; and every non-test `.go` file of the module, where `goLiteralHits` counts only the value of a string literal (`*ast.BasicLit` of kind `STRING`, read through `File.AST`), so a comment naming the tool is no hit.
It applies `scankit.RequireFloor` to each surface and fails listing every hit with file and line.
Recorded provider transcripts (`.jsonl`) under `testdata` are output a provider wrote, not prompts lym sends, and are not scanned.
The tree passes once cards 3 and 12 have landed: `holdNotice` holds the only Go literal naming the tool today, and the seat smoke fixtures the only `testdata` prompts.

`PATTERN.md` gains this line under `## Agents and prompts`, after `PATTERN-parent-directive`:

```
- `PATTERN-agent-messaging` — Messaging another session from an agent: only through creel, never an engine's own tool, bar a fork addressed in-session under ForkSubagents. (test)
```

**ImpactSummary:** One new tier0 scan test and one PATTERN entry line; no production code changes.
